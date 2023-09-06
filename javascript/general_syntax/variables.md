---
layout: page
title: "Variables"
permalink: /javascript/variables
---

## Variable names

* Variables must begin with a letter or dollar sign `$` or underscore `_`
* Its best practice in JavaScript to name variables in camelCase for mutable variables and upper case for constant ones.

## Hoisting

[comment]: <> (TODO: Verify this section content and see if strict mode fixes this.)

Hoisting is (to many developers) an unknown or overlooked behavior of JavaScript.  In JavaScript, a variable can be declared after it has been used.  In other words; a variable can be used before it has been declared.  Hoisting is JavaScript's default behavior of moving all declarations to the top of the current scope.  JavaScript only hoists declarations, not initializations.

While the below example is technically valid JavaScript code y will be undefined when used in `elem` as it has been declared but not initialized.

```javascript
var x = 5; // Initialize x

elem = document.getElementById("demo"); // Find an element
elem.innerHtml = x + " " + y;  // Display x and y

var y = 7; // Initialize y

```

## Declaring variables

Use `const` if the variable value or type (for Arrays and Objects) should not be changed.  Variable declared using const are read only.  Use `let` for mutable variables.

***Note***: Objects (including arrays and functions) assigned to a variable using const are still mutable.  Using const declaration only prevents reassignment of the variable identifier.

```javascript
const x = 5;
const y = 7;
const z = x + y;
let a = z + x;
```

Do not use the `var` to declare variables. Var remains in JavaScript for legacy support.  `var` is used to declare variables are either globally of functionally scoped and do not support block level scope.  This means that if a variable is declared inside an if statement or a for loop it can be accessed outside the block and lead to unintended bugs.

## Declared but unassigned variables

When a javascript variable is declared it has an initial value of `undefined`.  If you do a mathematical operation on an undefined variable your result will be a `NaN` which means "Not a Number".

If you concatenate a string with an undefined variable you will get a result string of undefined.