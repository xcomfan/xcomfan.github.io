---
layout: page
title: "JavaScript Number"
permalink: /javascript/number
---

## Incrementing numbers

Numbers in JavaScript can be incremented and decremented with the `++`, `--`, `+=`, `-=` `*=` operators.

## Floating point numbers

To declare a decimal/floating point number use `const myDecimal = 4.2;`  All operations work the same on decimals as on integers. For example `const quotient = 4.4 / 2.2;`

### Printing floating point numbers

There are build in methods in the Numbers class that can help with printing floats to a set precision.

```javascript
let n = 123456.789
n.toFixed(0) // "1234567"
n.toFixed(2) // "123456.79"
n.toPrecision(4) // "1.235e+5"
```

## Get number from a string

Your can use the `parseInt()` function to get an integer from a string.  `const a = parseInt("007");`

You can also pass a radix argument (what base to use such as binary) `const a = parseInt("11",2);`

You can also use the `parseFloat()` function if working with floating point numbers.

## Math class operations

In addition to basic arithmetic operators such as +,-,/, and % there are useful function constants defined in the math object.  Below are examples of some useful ones.

```javascript
Math.pow(2,53) // 2 to power of 53
Math.pow(3, 1/3) // The cube root of 3
Math.round(.6) // 1.0 round to the nearest integer
Math.ceil(.6) // 1.0 round up to nearest integer
Math.floor(.6) // 0: round down to the nearest integer
Math.abs(-5) // 5: Absolute value
Math.max(x,y,z) // Return the largest argument
Math.min(x,y,z) // Return the smallest argument
Math.random() // Pseudo random number between 0 and 1.0
Math.sqrt(3) // The square root of 3
```

### Generating a Random Number

JavaScript has a Math.random() function that generates a random decimal number between 0 (inclusive) and 1 (exclusive) thus Math.random() can return a 0 but never return a 1.

To generate a random number between 0 and 19 

```javascript
Math.floor(Math.random() * 20); // Note the 20 not 19 since Math.random() will never return 1.
```

## Division by Zero

Division by Zero is not an error in JavaScript.  It will return `Infinity` or negative `Infinity` zero divided by zero will return a `NaN`
