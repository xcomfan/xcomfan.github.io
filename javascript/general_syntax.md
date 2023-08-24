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
* Its best practice in JavaScript to name variables in camelCase for mutable variables and upper case for constant ones.

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

Use const if the variable value or type (for Arrays and Objects) should not be changed.  Variable declared using const are read only.  

***Note***: Objects (including arrays and functions) assigned to a variable using const are still mutable.  Using const declaration only prevents reassignment of the variable identifier.

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

### Declared but unassigned variables

When a javascript variable is declared it has an initial value of `undefined`.  If you do a mathematical operation on an undefined variable your result will be a `NaN` which means "Not a Number".

If you concatenate a string with an undefined variable you will get a result string of undefined.

## Data types

### String

Strings are defined using either single or double quotes.

`const myStr = 'This is my string'` or `const myStr = "This is my string"`

`"` and `'` characters in a string can be escaped with a `\`

JavaScript strings are immutable.  This means that once they are created they cannot be altered.  The only way to change a declared string is to assign it a new value.  

For example the code below is **NOT** valid

```javascript
let myStr = "Bob";
myStr[0] = "J";
```

The code below is correct however.

```javascript
let myStr = "Bob";
myStr = "Job";
```

You can however use array notation to index into strings.

```javascript
const firstName = "Boris";
const firstLetter = firstName[0]
```

Strings can be concatenated with the `+` and `+=` operators.

```javascript
const ourStr = "I come first. " + "I come second."

const anAdjective = "awesome!";
let ourStr = "Pizza is";
outStr += anAdjective;
```

You can also concatenate a string as a mix of constants and variables.  For example `const ourStr = "Hello" + ourName + ", how are you?"`

You can use the `.length` property to get the length of a string `console.log("Some Text".length)`

To check if a string is empty use the expression `if (value === "")`

With Javascript ES6 you can create stings from template literals.  This is done by using the back-quote to surround the strings and the `${}` syntax for template values.

```javascript
const person = {
  name: "Boris"
  age: 40
};

const greeting = `Hello my name is ${person.name}.  
I am ${person.age} years old.`;

// Will print the greeting to console on two lines as we have newline in the template string.
console.log(greeting);
```

### Numbers

Numbers in JavaScript can be incremented and decremented with the `++`, `--`, `+=`, `-=` `*=` operators.

To declare a decimal/floating point number use `const myDecimal = 4.2;`  All operations work the same on decimals as on integers. For example `const quotient = 4.4 / 2.2;`

Your can use the `parseInt()` function to get an integer from a string.  `const a = parseInt("007");`

You can also pass a radix argument (what base to use such as binary) `const a = parseInt("11",2);`

### Booleans

Booleans may be one of 2 values `true` or `false`.  Not that Boolean value are ever written in quotes.  `"true"` and `"false"` have no special meaning in JavaScript.

### Objects

#### Freezing an object

If you want to make sure that an object does not change you need to use the `Object.freeze` functionality.

[comment]: <> (TODO: Need to test this code.)

```javascript
let obj = {
  name: "Southgate Hauling Service",
  review: "Awesome"
}

Object.freeze(obj);
obj.review = "bad"; // Will return an error
obj.newProp = "Test"; //will return an error

```

## Comparison Operators

### Strict equality operator (== vs ===)

`===` is the strict equality operator in JavaScript.  `==` is the equality operator.  The difference between the to is `==` will try to convert both types being compared to a common type, `===` does not do the conversion.  If the values being compared with a `===` operator are not the same type they are considered unequal and the strict equality operator will return `false`.

### Inequality and strict inequality operators

```javascript
3 !== 3 // false
3 !== '3' // true
4 !== 3 // true

1 != 2 // true
1 != "1" //false
1 != 1 // false
0 != false // true
```

### Greater and less than operators

The `<`, `<=`, `>`, `>=` operators all do not have a strict counterpart and will try to convert to a common type for the comparison.

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

## Logical Operators

### Truthy and falsy in JavaScript

In JavaScript a **Truthy** value is a value that is considered `true` when encountered in a Boolean context.  All values are truthy unless they are defined as falsy.  In other words all value are truthy except `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined` and `NaN`.

Below are examples of *truthy* values which would be coerced to true by type coercion if executed in an if block.

```javascript
if (true)
if ({})
if ([])
if (42)
if ("0")
if ("false")
if (new Date())
if (-42)
if (12n)
if (3.14)
if (-3.14)
if (Infinity)
if (-Infinity)
```

### Logical AND

Logical AND operator is written as `&&` in JavaScript.  For example `a > 0 && b > 0`

#### Logical AND assignment

The logical AND assignment `&&=` operator only evaluates the right operand and assigns to the left if the left operand is truthy.

```javascript
let a = 1;
let b = 0;

a &&= 2;
console.log(a); // expected output 2

b &&= 2;
console.log(b); // expected output 0
```

### Logical OR

Logical OR operator is written as `||` in JavaScript.  For example `a > 0 || b > 0`

#### Logical OR assignment

The logical OR assignment `||=` operator only evaluates the right operand and assigns to the if the left operand is falsy

```javascript
const a = { duration: 50, title ''};

a.duration ||= 10;
console.log(a.duration); // expected output 50

a.title ||= 'title is empty.';
console.log(a.title); // expected output "title is empty"
```

### Logical NOT

The logical NOT is operator is written as `!` in JavaScript.  For example `!(a > 0 || b > 0)`

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

for (let i = 0, len = cars.length, text = ''; i < len; i++){
    text += cars[i] + "<br>"
}

let i = 0;
let len = cars.length;
let text = "";
for (; i < len; ){
    text += cars[i] + "<br>";
    i ++;
}
```

### for in

The `for in` statement loops through the properties of an object.

```javascript
const person = {fname: "John", lname: "Doe", age:25}

let text = "";
for (let x in person){
  text += person[x]
}

const numbers = [45, 4, 9, 16, 25]

let txt = "";
for (let x in numbers){
  txt += numbers[x]
}
```

### for of

The JavaScript `for of` statement loops through the values of an iterable object.  It lets you loop over iterable data structures such as Arrays, Strings, Maps, NodeLists and more.

#### Looping over an array

```javascript
const cars = ["BMW", "Volvo", "Mini"];

let text = "";
for (let x of cars) {
  text += x;
}
```

#### Looping over a string

```javascript
let language = "JavaScript";

let text = "";
for (let x of language) {
text += x;
}
```

### Array.forEach()

[comment]: <> (TODO: This should probably be moved to the array section once I have that)

The `forEach()` method calls a function (a callback function) once for each array element.

Note the callback function we are writing takes 3 values.
* The item value
* The item index
* The array itself

```javascript
const numbers = [45, 4, 9, 16, 25];

let txt = "";
numbers.forEach(myFunction);

function myFunction(value, index, array){
  txt += values;
}
```

The example above only uses the value parameter.  It can be rewritten as...
```javascript
function myFunction(value){
  txt += value;
}
```

## Ternary Operator

The syntax for a ternary operator in JavaScript is a ? b : c, where a is the condition, b it the code tor un when the condition returns true, and c is the code to run when the condition returns false.

```javascript
function checkEqual(a,b){
  return a === b ? "Equal" : "Not Equal";
}
```

You can also nest ternary operators

```javascript
function findGreaterOrEqual(a,b){
  return (a === b) ? "a and b are equal"
    : (a > b) ? "a is greater"
    : "b is greater";
}
```

Its considered best practice to format conditional operators such that each condition is on a separate line as in the example above.  
