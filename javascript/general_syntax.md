---
layout: page
title: "JavaScript Reference"
permalink: /javascript/general_syntax
---

## General information

* JavaScript is case sensitive
* HTML is not case sensitive: It is an accepted practice to write HTML in all lowercase inside of JavaScript.  For example in HTML its accepted to use `onClick`, but in JavaScript you would use `onclick`.
* JavaScript does automatic garbage collecting
* Variables are untyped and JavaScript will try to convert between types if used that way in code.
* Statements are executed.  Expressions are evaluated to produce a value.

## Comments

`// for single line comment`

```javascript
/*
multi line comment
*/
```

## Variable names and declaration

### Variable names

* Variables must begin with a letter or dollar sign `$` or underscore `_`
* Declaration of variables id done with the `var` keyword

### Declaring with let

```javascript
let j; // one simple variable
let j=0; // one variable one value
let p,q; // two variables
let x = 2.34, y = Math.cos(0.75), r, theta // many variables
let x = 2,
    f = function(x) { return x*x },
    y = f(x) // each declared on its own line
```

### Declaring with const

Use const if the variable value or type (for Arrays and Objects) should not be changed.

```javascript
const x = 5;
const y = 7;
const z = x + y;
```

### Declaring with var

This is still supported, but you should not use it.  var is there for support of older pre 2015 browsers.  Only use this if you must.

```javascript
var j=0; // one variable one value
```

## Conditionals

### if

```javascript
if (username === null) username = "John Doe";

if (!address){
    address = "";
    message = "Please specify a mailing address.";
}
```

### if else

```javascript
if (hour < 18) {
  greeting = "Good day";
} else {
  greeting = "Good evening";
}
```

### switch

```javascript
switch (new Date().getDay()) {
  case 0:
    day = "Sunday";
    break;
  case 1:
    day = "Monday";
    break;
  case 2:
     day = "Tuesday";
    break;
  case 3:
    day = "Wednesday";
    break;
  case 4:
    day = "Thursday";
    break;
  case 5:
    day = "Friday";
    break;
  case 6:
    day = "Saturday";
}
```

## Loops

### while loop

Loops though a block of code as long as a specified condition is true.

```javascript
let count = 0;
while (count < 10){
    console.log(count);
    count++;
}
```

### do/while loop

Execute once and repeat as long as the condition is true.

```javascript
let text = ""
let i = 0;
do {
    text += i + "<br>";
    i++;
}
while (i < 5)
```

### for loop

```javascript
let text = "";
for (let i = 0; i < 5; i++>) {
    text += "The number is " + i + "<br>";
}
```

### for in

### for of
