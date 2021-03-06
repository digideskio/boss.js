# Boss.js

Automatically load balance asyncronous jobs across multiple processes in a round-robin fashion.

## When to use Boss

Let's say that you need to parse multiple URLs, or ping multiple machines for a given list of IPs. If you have a list of URLs or IPs, and you need to process them, Boss allows you to redistribute the work across several processes in a cyclical, round-robin, mode so that the list is entirely processed in multiple parallel jobs.

It's used in production at [Mashape](https://www.mashape.com) to process messages stored in a queue, like RabbitMQ or Amazon SQS.

## Installing Boss

```bash
npm install boss
```

## Usage

```javascript
var Boss = require('boss');

var execution = new Boss.Execution(function(context) {

  // Master process code. Use context.dispatch(message) to send a message to a job worker.
  context.dispatch({an:"object"});

}, function(message, next) {

  // Job worker code. The "message" argument is the message object to handle. Call next() when the operation has been completed.
  next();

}, options);

execution.start(); // Start the execution
```

### Options

* `debug`: Set to `true` to enable debugging console information.
* `jobWorkers`: The number of job workers to execute. This number doesn't count the *master worker*, that coordinates the child job workers. This means that the number of node processes will always be the number of `jobWorkers` + 1. If this option is not specified, the total number of CPU cores **-1** will be used.

### Example

```javascript
var Boss = require('boss');

var execution = new Boss.Execution(function (context) {
  // Define the master process code here
  var index = 0;
  setInterval(function () {
    context.dispatch({
      num: index++
    });
  }, 0);
}, function (message, next) {
  // Define the job worker code here
  console.log(process.pid + " received number " + message.num);
  next();
}, {
  debug: true,
  jobWorkers: 6
});

execution.start();
```

## Advanced

The `context` variable has two functions that you can use to control the submission flow:

* `context.getRunning()`: Get the number of the current running jobs across all the workers.
* `context.isAvailable()`: Ask if there is at least one idle worker that is ready to accept a job. This is useful if you don't want to redistribute all the jobs immediately to the workers, but you prefer to execute them once at a time (max number of concurrent jobs = number of workers).

For example:

```javascript
var Boss = require('boss');

var execution = new Boss.Execution(function (context) {
  // Define the master process code here
  var index = 0;
  setInterval(function () {
	if (context.isAvailable()) { // Only execute if there is at least one job worker available
      context.dispatch({
        num: index++
      });
    }
  }, 0);
}, function (message, next) {
  // We're simulating a 1s delay in the job worker execution
  setTimeout(function() {
    console.log(process.pid + " received number " + message.num);
    next();
  }, 1000);
});

execution.start();
```

## License

The MIT License

Copyright (c) 2015 Mashape (http://mashape.com)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
