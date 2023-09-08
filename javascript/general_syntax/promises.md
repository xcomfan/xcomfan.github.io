---
layout: page
title: "Javascript Promises"
permalink: /javascript/promises
---

[comment]: <> (TODO: Need to reivew update work though examples and test this section..)

A promise in JavaScript is exactly what it sounds like.  You use it to make a promise to do something, usually asynchronously.  Whe the task completes you either fulfill the promise of fail to do so.  

A promise is a constructor function, so you need to use the `new` keyword to create one.  It takes a function as its argument with two parameters - resolve and reject.  These are the methods to determine the outcome of the promise.  

A promise has 3 states:

* pending
* fulfilled
* rejected

```javascript
const myPromise = new Promise((resolve, reject) => {
  if (condition here){
    resolve("Promise was fulfilled");
  } else {
    reject("Promise was rejected");
  }
});
```

The example above uses strings for the arguments of these functions, but it can really be anything.  Often, it might be an object, that you would use data from, to put on your website or elsewhere.

Promises are most useful when you have a process that takes an an unknown amount of time in your code.  (i.e., something asynchronous) often a server request.

When you make a server request, it takes some amount of time, and after it completes you usually want to do something with the response from the server.  This can be achieved using the `then` method.  The then method is executed immediately when your promise is fulfilled with the `resolve` method.

```javascript
myPromise.then(result => {

});
```

`catch` is the method used when your promise has been rejected.  It is executed immediately after a promise's reject method is called.

```javascript
myPromise.catch(error => {

});
```
