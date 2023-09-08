---
layout: page
title: "JavaScript Number"
permalink: /javascript/number
---

## Incrementing numbers

Numbers in JavaScript can be incremented and decremented with the `++`, `--`, `+=`, `-=` `*=` operators.

## Floating point numbers

To declare a decimal/floating point number use `const myDecimal = 4.2;`  All operations work the same on decimals as on integers. For example `const quotient = 4.4 / 2.2;`

## Get number from a string

Your can use the `parseInt()` function to get an integer from a string.  `const a = parseInt("007");`

You can also pass a radix argument (what base to use such as binary) `const a = parseInt("11",2);`

## Generating a Random Number

JavaScript has a Math.random() function that generates a random decimal number between 0 (inclusive) and 1 (exclusive) thus Math.random() can return a 0 but never return a 1.

To generate a random number between 0 and 19 

```javascript
Math.floor(Math.random() * 20); // Note the 20 not 19 since Math.random() will never return 1.
```
