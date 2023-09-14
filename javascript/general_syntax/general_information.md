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
