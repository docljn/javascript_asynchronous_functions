## How to Create a Promise

```JavaScript
var promise = new Promise(function(resolve, reject) {
  // do a thing, possibly async, then…
  if (/* everything turned out fine */) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});
```
- the promise constructor takes a single argument: a callback with two parameters
```javascript
new Promise(/* executor*/ function (resolve, reject) { ... } );
```
- if everything worked, call resolve(your_success_value)
- if the result fails, call reject(your_fail_value)
- it is a good idea to 'reject' with an Error object, as the stack trace makes debugging easier

#### English: Mum's promised me a new phone

Currently, I don't know if I will get the phone or not.
- the promise is **pending**

There are two potential outcomes:
- I get the phone: the promise is **resolved**
- I don't get the phone: the promise is **rejected**

#### Javascript

```javascript
// Pre-existing conditional
const isMumHappy = false;

// Promise
const willIGetNewPhone = new Promise(
    function (resolve, reject) {
        if (isMumHappy) {
            const phone = {
                brand: 'Samsung',
                color: 'black'
            };
            resolve(phone); // fulfilled
        } else {
            const reason = new Error('mom is not happy');
            reject(reason); // reject
        }
    }
);
```

## How to Consume a Promise

**Will I get my new phone?**

- askMum() consumes the promise willIgetNewPhone

- then() **takes** two _optional_ arguments:
  - a callback for a success case
  - a callback for a failure case

- .then() **returns** a **new promise**



```javascript
const askMum = function () {
    willIGetNewPhone
        .then(function (fulfilled) {
            // yay, you got a new phone
            console.log(fulfilled);
         // output: { brand: 'Samsung', color: 'black' }
        })
        .catch(function (error) {
            // oops, mom don't buy it
            console.log(error.message);
         // output: 'mom is not happy'
        });
};
askMum();
```

### Chaining Promises

A function passed to "then" can return another promise, meaning you can chain the calls together.

You can also chain promises after a .catch i.e. if you want to specify the action that follows a failure.

This becomes really useful if the input for a subsequent function is the output of an earlier function...


#### Using Promise Chains

**You promise your friend that you will show them your new phone when your mum buys it.**

You can only start the showOff promise after the willIGetNewPhone promise (obviously!) because it depends on your having a new phone...
-  the input for "showOff" is the output of "willIgetNewPhone"


#### Create another promise
```javascript
// 2nd promise
const showOff = function (phone) {
    return new Promise(
        function (resolve, reject) {
            const message = 'Hey friend, I have a new ' +
                phone.color + ' ' + phone.brand + ' phone';

            resolve(message);
        }
    );
};
```
Because we're not using reject, we can simply call Promise.resolve to shorten the code
```javascript
// 2nd promise
const showOff = function (phone) {
    const message = 'Hey friend, I have a new ' +
                phone.color + ' ' + phone.brand + ' phone';

    return Promise.resolve(message);
};
```
#### Chain the Promises

```javascript
// call our promise
const askMum = function () {
    willIGetNewPhone
    .then(showOff) // chain it here
    .then(function (fulfilled) {
            console.log(fulfilled);
         // output: 'Hey friend, I have a new black Samsung phone.'
        })
        .catch(function (error) {
            // oops, mom don't buy it
            console.log(error.message);
         // output: 'mom is not happy'
        });
};
```

### Error Handling
- .catch(failureCallback) is short for .then(null, failureCallback);
- if you don't add .catch() any thrown errors will be swallowed, and you won't even see them in your console!
  - to avoid this, use this pattern:
  -
  ```javascript
  myPromise().then(function () {
    return anotherPromise();
  }).then(function () {
    return yetAnotherPromise();
  })).catch(console.log.bind(console);)
  ```


### Promise.all
**Promise.all(array_of_promises)** will only continue if all the promises in the array have been fulfilled.  This is _really useful_!

Use Promise.all where you would habitually reach for forEach(), for or while.
All those functions return undefined, which means they cannot be used in a promise chain.

Promise.all() takes an array of promises as input, and returns another promise that only resolves when every one of those other promises has resolved. It is the asynchronous equivalent of a for-loop.

Promise.all() also passes an array of results to the next function, which can get very useful, for instance if you are trying to get() multiple things from a database. The all() promise is also rejected if any one of its sub-promises are rejected, which is even more useful.

```JavaScript
Promise.all([img1.ready(), img2.ready()]).then(function() {
  // all loaded
}, function() {
  // one or more failed
});
```

## Using Promises In Node.js
```
> npm install promise --save
```
```javascript
const Promise = require("promise");
```

The "promise" library also provides a couple of really useful extensions for interacting with node.js
- Promise.denodeify()
  - takes a function which would normally return a callback, and returns a promise instead
  -
```javascript
var readFile = Promise.denodeify(require('fs').readFile);
```
- function.nodeify(callback)
  - is chained onto the end of a series of .then() functions
  - if a callback is provided
    - call with (error, result) and return "undefined"
  - if no callback is provided
    - return the promise
  -
```JavaScript
function readJSON(filename, callback){
  return readFile(filename, 'utf8')
    .then(JSON.parse)
    .nodeify(callback);
}
```

## Using Promises with a callback-based API
Use the [revealing constructor pattern](https://blog.domenic.me/the-revealing-constructor-pattern/) to wrap a callback-based API.

### setTimeout() - a simple example
#### Original code: using a callback
```javascript
function doStuff() {/*...*/};
setTimeout(doStuff, 300);
```

#### We want a new Promise function
- the Promise constructor takes a single function as an argument
- that function takes two arguments:
  - a resolve function
  - a reject function
  - _both resolve and reject are callback functions_
-
```javascript
function timeout(delay) {
  return new Promise(function (resolve, reject) {
    setTimeout(resolve, delay);
  });
};
```
- setTimeout doesn't provide any information for an error state, so we can't use the _reject_ callback
- resolve is passed as the callback to setTimeout
- our new timeout() function returns a Promise

There are also in-depth discussions on [Stack Overflow](https://stackoverflow.com/questions/22519784/how-do-i-convert-an-existing-callback-api-to-promises) and [benmccormick.org](https://benmccormick.org/2015/12/30/es6-patterns-converting-callbacks-to-promises/).


## Modern JavaScript
### ES6 and fat arrow functions

The willIGetNewPhone example looks like this in ES6:
```javascript
/* ES6 */
const isMumHappy = true;

// Promise
const willIGetNewPhone = new Promise(
    (resolve, reject) => { // fat arrow
        if (isMumHappy) {
            const phone = {
                brand: 'Samsung',
                color: 'black'
            };
            resolve(phone);
        } else {
            const reason = new Error('mom is not happy');
            reject(reason);
        }

    }
);

const showOff = function (phone) {
    const message = 'Hey friend, I have a new ' +
                phone.color + ' ' + phone.brand + ' phone';
    return Promise.resolve(message);
};

// call our promise
const askMum = function () {
    willIGetNewPhone
        .then(showOff)
        .then(fulfilled => console.log(fulfilled)) // fat arrow
        .catch(error => console.log(error.message)); // fat arrow
};

askMum();
```

For more on the fat arrow, you can visit [StrongLoop]( https://strongloop.com/strongblog/an-introduction-to-javascript-es6-arrow-functions/) and [IBM's Developer Site](
https://developer.ibm.com/node/2015/09/21/an-introduction-to-javascript-es6-arrow-functions/)

### ES7 Async Await
This makes the syntax even prettier, but that's for another time...
<https://developers.google.com/web/fundamentals/primers/async-functions>
<https://pouchdb.com/2015/03/05/taming-the-async-beast-with-es7.html>
