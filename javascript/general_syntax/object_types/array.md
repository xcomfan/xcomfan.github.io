---
layout: page
title: "JavaScript Arrays"
permalink: /javascript/arrays
---

## Adding and removing data to an array

To append data to the end of an array use the `push()` function.  To remove data from the end (and get the value) use the `pop()` function.  To remove value from front of array and get its value use the `shift()` function.  To insert a value at the beginning of an array use the `unshift()` function.

```javascript
const arr1 = [1,2,3];
arr1.push(4)

let last = arr1.pop()
let first = arr1.shift()

arr1.unshift(last);
```

To get the length of an array use the length property. `len = myArr.length`

## Using the spread operator

Spread operator was added in ES6.  It allows us to expand arrays and other expressions in place where multiple parameters or elements are expected.

The spread operator is denoted with the tripple dot `...` syntax, and it returns an unpacked array.  

```javascript
const arr = [6, 89, 3, 45];
const maximum = Math.max(...arr)
```

The spread operator only works in place, like in an argument to a function or in any literal.  The below code is will now work.

```javascript
const spread = ...arr;
```

## Destructuring Arrays

The spread operator unpacks all contents of an array in to a comma separated list, but it does not let you pick and choose which elements of that list you assign to variables.  Destructuring an array lets us do that.

```javascript
const [a, b] = [1,2,3,4,5,6];
console.log(a, b); // Will log a and b as 1 and 2

const [a, b,,, c] = [1,2,3,4,5,6];
console.log(a,b,c); // Will log a b and c as 1, 2 and 5 (,, skipped 2 entries)
```

You can also collect results into a separate array...

```javascript
const [a, b, ...arr] = [1,2,3,4,5,7];
console.log(a,b); // Will display as 1, 2
console.log(arr); // Will display [3,4,5,7]
```