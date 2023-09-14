---
layout: page
title: "JavaScript Undefined and Null"
permalink: /javascript/undefined_and_null
---

In JavaScript null is "nothing".  It is supposed to be something that doesn't exist.  Unfortunately, in JavaScript, the data type of null is an Object.

It should be considered a bug that `typeof` for null is an Object.

You can empty an object by setting it to `null``

```javascript
var person = {firstName: "John", lastName: "Doe", age: 50};
person = null; // Now the value is null but type is an object.
```

The difference between undefined and null is that null is an Object and `typeof` undefined is undefined.  In other words `null` is for objects and `undefined` is for variables, properties and methods.

Both `null` and `undefined` are falsy values.
