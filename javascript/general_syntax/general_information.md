---
layout: page
title: "General Information"
permalink: /javascript/general_information
---

JavaScript is case sensitive

HTML is not case sensitive: It is an accepted practice to write HTML in all lowercase inside of JavaScript.  For example in HTML its accepted to use `onClick`, but in JavaScript you would use `onclick`.

JavaScript does automatic garbage collecting.

Variables are untyped and JavaScript will try to convert between types if used that way in code.

Statements are executed.  Expressions are evaluated to produce a value.

## Strict Mode

* In strict mode all variables must be declared.
* Functions invoked as functions and not as methods have a this value of undefined.  In non strict mode functions invoked as functions are passed the global object for the this value.
* When `call()` or `apply()` is used, the value is exactly the value passed as the first argument to `call()` or `apply()`.  In non strict mode null and undefined values are replaced with the global object and non object values are converted to objects.
* In strict mode the arguments object in a function holds a static copy of the values passed to the function.  In non strict mode the arguments object has "magical" behavior in which elements of the array and named function parameters both refer tot he same value.




## Definitions

[comment]: <> (TODO: These are from my older notes.  Need to review in merge in where they make sense.)

Expression - is a phrase of JavaScript that a JavaScript interpreter can evaluate to produce a value.  A constant embedded literally in your program is a very simple kind of expression.  A variable name is also a simple expression that evaluates to whatever value has been assigned to that variable.  x * y is another example of an expression.  So is array indexing and function invocation.

Higher Order Function - is a function that operates on functions taking one or more functions as arguments and returning a new function.  For example you can have a function that wraps a function that returns true or false.  This will be a not function.

Host Object - an object defined by the host environment (such as a web browser) within which the JavaScript interpreter is embedded.

dentifier - a variable name 

Literal is a data value that appears directly in a program.  For example…
* 12
* 1.2
* "hello world"
* true
* null

Module - A single file of JavaScript code.

Native Object - Objects defined by ECMA specification (Arrays, Date, Functions, Regular Expressions)

Numeric Literal - When a number appears directly in a JavaScript program.

Own Property - Property defined directly on an object.