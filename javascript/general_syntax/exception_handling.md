---
layout: page
title: "Javascript Exception Handling"
permalink: /javascript/exceptions
---

## throw

`throw` expression

The expression above can evaluate to any time.  You can throw a number that represents an error code or a human readable string.

When an expression is thrown in JavaScript the interpreter stops normal program execution and jumps to the nearest exception handler.

## try catch finally

```javascript
try {
  // some coe that will run unless there is a problem.
}
catch(e){
  // Statements in this block are executed if and only if the try block throws an exception.
  // e in this example refers to the error thrown.
}
finally{
  // This block is always executed.
}
```