# Asynchronous JavaScript and Promises

## Why do we need asynchronous JavaScript functions?

JavaScript is single threaded: in the browser, user actions like highlighting text are in the same queue as updating styles, and the same queue as JavaScript. A delay in any one thing in a queue will delay all the others.

As a human, the only similar single-threaded activity is a sneeze: everything else stops...

### One solution: Events and Callbacks

If you add an event listener to an element of your page, JavaScript can stop executing until the listener is called.

```JavaScript
var img1 = document.querySelector('.img-1');

img1.addEventListener('load', function() {
  // woo yey image loaded
});

img1.addEventListener('error', function() {
  // argh everything's broken
});
```

But, a simple event listener doesn't allow for events that happen before the event listener starts listening for them, so we have to use something like .complete for images, or "DOMContentLoaded" for documents to start the script.

```JavaScript
var img1 = document.querySelector('.img-1');

function loaded() {
  // woo yey image loaded
};

if (img1.complete) {
  loaded();
}
else {
  img1.addEventListener('load', loaded);
};

img1.addEventListener('error', function() {
  // argh everything's broken
});
```

But that **still** doesn't catch an image which errored before we started listening...  We have no access to the call stack, and we're stuck.   

For more detail, see [How does callback work in JavaScript at scotch.io](https://scotch.io/@pratikpandey/how-does-callback-work-in-javascript) which includes an excellent diagram of how JavaScript handles callbacks and why you lose access to the stack.

### Is there a better way?

Events are useful where you don't care what happened before you attached the listener: keyup, mousedown, onclick etc.

However, events don't work at all well when you need to know if something worked or not aka async success/failure.


In that situation, what you would like to call might look a bit like this:

```JavaScript
img1.callThisIfLoadedOrWhenLoaded(function() {
  // loaded
}).orIfFailedCallThis(function() {
  // failed
});

// and…
whenAllTheseHaveLoaded([img1, img2]).callThis(function() {
  // all loaded
}).orIfSomeFailedCallThis(function() {
  // one or more failed
});
```


## Synchronous and Asynchronous Code Compared: Callbacks vs. Promises
From YouTube: Redemption from Callback Hell, 2013

**Promises give you back the call stack**
- callbacks mean you have no catch, no throw, no return...
- all of the synchronous operations are now available as a result of Promises
- you also have the guarantee that the function within a promise will only be called once

### Return, Catch, Throw, ReThrow compared
- synchronous:
```javascript
const user = getUser("lnoble");
const name = user.name;
```

- with a callback:
```javascript
getUser("lnoble", function (error, user) {
  // ... database or API call to get user
});
```

- What would a promise look like? (we're building up how to use a Promise as we go along here...)
```javascript
getUser("lnoble").then( function (user) {
  // .. something which fulfils the promise
}, function () {
  // .. error handling equivalent
});
```
- then takes two arguments
1. fulfilment handler function: the value of the promise ~= return
2. rejection handler function: the reason for which the promise is not valid ~= throw
- once either function has been called, any future call to then will not be executed

- from within then you can
  1. return another promise
  2. return a synchronous value, (or undefined but try to avoid that)
  3. throw a synchronous error


- **Example type 2** so, what if the function getUser() returns a _Promise_, and you want to return the result? "Return Equivalent"
- in this case "user" is the result of the promise
```javascript
getUser("lnoble").then( function (user) {
  return user.name;
  // promise fulfilled
  // just like returning normally from a function
  // you get the user as an object returned from the function
} );
```

- **Example type 3** what about error handling? "Throw Equivalent"
- synchronous code:
```javascript
const user = getUser("lnoble");
if (!user) throw new Error("no user!");
const name = user.name;
```
- Promise equivalent:
```javascript
getUser("lnoble").then( function (user) {
  if (!user) throw new Error("no user!");
  return user.name;
});
```

- what about error handling? "Catch Equivalent"
- synchronous code:
```javascript
try {
  deliverTweetTo(tweet, "lnoble");
} catch error {
  handleError();
};
```
- Promise equivalent:
```javascript
deliverTweetTo(tweet, "lnoble")
.then(undefined, handleError);
// we haven't defined what we want to do if delivery is successful
```

- **Another example type 3** and how about "Rethrow" i.e. catch an error, and use it to throw a new error - a compound error
- synchronous code:
```javascript
try {
  const user = getUser("lnoble");
} catch (error) {
  throw new Error("ERROR: " + error.message);
}
```
- promise equivalent:
```javascript
getUser("lnoble").then( undefined, function (error) {
  throw new Error("ERROR: " + error.message);
});
```

### Synchronous, Callback and Promise compared

#### Happy Path
- synchronous
```JavaScript
const user = getUser("lnoble");
const tweets = getNewTweets(user);
updateTimeline(tweets);
```

- using callbacks
```JavaScript
getUser("lnoble", function (user) {
  getNewTweets(user, function (tweets) {
    updateTimeline(tweets);
  });
});
```

- using promises
```JavaScript
getUser("lnoble")
// getUser returns an object: a promise
// that value is passed to getNewTweets
// and so on
.then(getNewTweets)
.then(updateTimeline);
```

#### Handling Exceptions

- synchronous
```JavaScript
try {
  const user = getUser("lnoble");
  const tweets = getNewTweets(user);
  updateTimeline(tweets);
} catch (error) {
  handleError(error);
}
```

- using callbacks (_oh help!_) we get an unholy mess
```JavaScript
getUser("lnoble", function (error, user) {
  if (error) {
    handleError(error);
  } else {
    getNewTweets(user, function (error, tweets) {
      if (error) {
        handleError(error);
      } else {
        updateTimeline(tweets, function (error) {
          if (error) handleError(error);
        });
      }
    });
  }
});
```
- that is not pretty
- we have to handle each error manually
- we don't have a call stack when we step into a callback, so we can't throw errors
- if you do try to throw an error within a callback, you end up with an "uncaughtException" event, and a crashed server!

- using promises
- a reminder of that synchronous code:
```JavaScript
try {
  const user = getUser("lnoble");
  const tweets = getNewTweets(user);
  updateTimeline(tweets);
} catch (error) {
  handleError(error);
}
```
- and the new pattern: _good grief isn't that better!_
```JavaScript
getUser("lnoble")
.then(getNewTweets)
.then(updateTimeline);
.then(undefined, handleError);
```
- if _any_ of the steps in a promise chain fails, the error propagates all the way to the "catch" equivalent handleError without calling any subsequent function.

## So WTF is a promise?

**A promise is a _placeholder_ representing the eventual result (value) of an asynchronous operation**
- the promise placeholder will be replaced by the result value (if successful) or reason for failure (if unsuccessful)

**If you don't need to know _when_ something happened, but just _whether_ it happened or not, then a promise is what you are looking for.**

This would be particularly useful if you were, say, fetching data in various ways or from various places, and could only act on the data once it was all in place.

A promise is a bit like an event listener, except that:
- a promise can only succeed or fail once
- a promise can't switch from fail to success, or vice versa
- once you have a result, the promise is immutable
- if a promise has succeeded or failed, and you **later** add a success/failure callback, the correct callback will be called
- it doesn't matter that the event occurred **before** you added the callback

Always return a result from a function inside a Promise, otherwise there's nothing for the subsequent function to act on.





## Promise Terminology
A promise can be:
- fulfilled: The action relating to the promise succeeded
  - the asynchronous operation has completed
  - the promise has a _value_
  - the promise will not change again
- rejected: The action relating to the promise failed
  - the asynchronous operation failed
  - the promise will never be fulfilled
  - the promise has a _reason_ indicating why the operation failed
  - the promise will not change again
- pending: Hasn't fulfilled or rejected yet
  - the asynchronous operation hasn't completed yet
  - can transition to fulfilled or rejected

A promise can also be referred to as:
- settled: Has been fulfilled _or_ rejected and is thus _immutable_


## Browser support & Compatibility
In Chrome 32, Opera 19, Firefox 29, Safari 8 & Microsoft Edge, promises are enabled by default.
If you want to use them anywhere else, e.g. older browsers, or Node.js, there is a polyfill called [lie](https://github.com/calvinmetcalf/lie) but see also the [ES6-Promises Readme](https://github.com/jakearchibald/ES6-Promises#readme)
Anything with a _then()_ method is treated as promise-like or _thenable_.
  - any library that returns a Q promise
  - jQuery's deferreds _if_ they are cast to a standard promise: otherwise they are *not* promises at all!

## Next: Using Promises
[Creating and Using  Promises](./creating_promises.md)

## References

API reference containing all methods:
<https://developers.google.com/web/fundamentals/primers/promises#promise-api-reference>

Promises Spec: <https://github.com/promises-aplus/promises-spec>

From Scotch.io:
<https://scotch.io/tutorials/javascript-promises-for-dummies>
<https://scotch.io/@pratikpandey/how-does-callback-work-in-javascript>

From MDN:
<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises>

From Google:
<https://developers.google.com/web/fundamentals/primers/promises>

From Spring:
<https://spring.io/understanding/javascript-promises>

From Lindesay:
<https://www.promisejs.org>

From YouTube: Redemption from Callback Hell, 2013
<https://youtu.be/hf1T_AONQJU>

Common mistakes when using promises, and advanced Promise Wrangling:
<https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html>

The Revealing Constructor Pattern:
<https://blog.domenic.me/the-revealing-constructor-pattern/>
