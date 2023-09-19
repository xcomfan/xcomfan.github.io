---
layout: page
title: "JavaScript Arrays"
permalink: /javascript/arrays
---

## General Info
[comment]: <> (TODO: Need to validate this and generally everything on this page as a lot of it is from very old notes.)
Array indexing is 32 bit based so the largest possible value for an index is 2^32=2 

## Declaring an array

```javascript
let myArray = [] // An empty array no expression inside brackets means no values

let matrix = [[1,2,3],[4,5,6],[7,8,9]];

let a = new Array(); // declare using Array constructor
let a = new Array(10); // specify a length of 10 
```
[comment]: <> (TODO: Test our what new Array(10) will initialize the values to)

### Sparse arrays

You can have an array that has values omitted.  `let sparseArray = [1,,,,5];`  The omitted values will have value of undefined.


## Adding and removing data to an array

To append data to the end of an array use the `push()` function.  To remove data from the end (and get the value) use the `pop()` function.  To remove value from front of array and get its value use the `shift()` function.  To insert a value at the beginning of an array use the `unshift()` function.

```javascript
const arr1 = [1,2,3];
arr1.push(4)

let last = arr1.pop()
let first = arr1.shift()

arr1.unshift(last);
```

You can also use the delete operator.

```javascript
let a = [1,2,3];
delete a[1];
```

### splice method

`splice()` is the general purpose method for inserting, deleting or replacing array elements.  It will update length and shift array elements to higher or lower indexes as needed.

The first argument to splice specifies the index at which insertion or deletion occurs.  The second argument specifies the number of elements that should be deleted.  If second argument is omitted all elements after the fist argument are deleted.

`splice()` returns an array of the deleted elements or an empty array if no elements were deleted.  

The first two arguments can be followed by any number of additional arguments that specify elements to be inserted into the array starting at the position specified by the first argument.

```javascript
let a = [1,2,3,4,5,6,7,8];
a.splice(4); // returns [5,6,7,8]; a is [1,2,3,4]

let a = [1,2,3,4,5];
a.splice(2,0,'a','b') // returns [] a is [1,2,"a","b",3,4,5]
a.splice(2,2,[1,2],3) // returns ['a','b']; a is [1,2,[1,2],3,3,4,5]
```

## Getting length of an array.

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

## Useful array methods

### Converting array elements to a string with join()

`join()` converts all the elements of an array to strings and concatenates them returning the resulting string.  You can specify an optional string that separates the elements in the resulting string.

[comment]: <> (TODO: Test to check if comma is the default if no separator is specified)

```javascript
let a = [1,2,3];
a.join() // returns "1,2,3"
a.join(" ") // returns "1 3 4"
```

### reverse()

Reverses the order of the elements of an array and returns the reversed array.  It does this in place a new array is not created.

[comment]: <> (TODO: Test to verify if it is in place and if array is returned or if it just reverses the array)

### sort()

Sorts the elements of an array in place and returns the sorted array.  If sort() is called with no arguments it sorts the array elements in alphabetical order based on what they convert to string as.  Undefined elements are sorted to the end of the array.  

To sort in non alphabetical order pass a comparison function.  If the first argument of the compare function should appear before the second return a number less than 0 if after more than 0 if tye are the same return 0.

```javascript
let a = [33, 4, 1111, 222];
a.sort() // alphabetical order: 1111, 222, 33, 4

a.sort(function(a,b){return a-b;}); // reverse numerical order 
```

### concat()

Creates and returns a new array that contains elements of the original array on which `concat()` was invoked followed by each of the arguments to `concat()`.

```javascript
let a = [1,2,3];
a.concat([4,5],[6,7]) // returns [1,2,3,4,5,6,7]
```

### slice()

`slice()` returns a slice or sub array.  Its two arguments specify the start and end of the slice.  Includes the first element and all subsequent elements up to but not including the second element.  If just one argument is specified it returns all elements from the start position.  Negative value count from the back of the array.

```javascript
let a = [1,2,3,4,5]
a.slice(1,-1) // returns [2,3,4]
```

## Build in Functions for iterating over arrays

[comment]: <> (TODO: May want to move this and check if its not just arrays but iterables like in Python)

### forEach()

`forEach()` iterates over an array invoking a function you specify on each method.

```javascript
let data = [1,2,3,4,5];
let sumOfSquares = 0;
data.forEach(function(x){
    sumOfSquares + x*x
});
```

### map()

`map()` passes each element of the array on which it is invoked to the function you specify and returns an array containing the values returned by that function.  `map()` returns a new array.  It does not modify the array it was called on.

```javascript
a = [1,2,3];
b = a.map(function(x){return x*x}); // b will be [1, 4, 9]
```

### filter()

The `filter()` method returns an array containing a subset of elements of the array on which it is invokes.  The function passes to filter() needs to return true or false, and filter will iterate over the arrays similar to `forEach()`  It will return an array where the function evaluates the array values to true.

```javascript
let a = [5,4,3,2,1];
let smallvalues = a.filter(function(x){return x < 3>}); // [2,1]
let everyother = a.filter(function(x){return i%2 == 0}); // [5,3,1]
```

### every() and some()

The `every()` method is like a for all.  It returns true if and only if your predicate function returns true for all elements in the array.  `some()` is like a there exits.  It returns true if there exists at least one element in the array for which the predicate function returns true.   Both `every()` and `some()` stop iterating as soon as they have an answer.

```javascript
let a = [1,2,3,4,5];
a.some(function(x){ return x%2 == 0; }); // returns true because there are some even numbers
a.every(function(x){ return x < 10; }); // returns true all value are less than 10.
```

### reduce() and reduceRight()

`reduce()` takes two arguments.  One is a function that has to take two values and somehow make one value out of them.  The second is an option argument that is an initial second value passed to the function.  If no initial value is passed reduce will take the first two values of the array.

```javascript
let a = [1,2,3,4,5];
let sum = a.reduce(function(x,y){ return x + y},0) // sum of values
let product = a.reduce(function(x,y){ return x * y },1) // product ov values
let max = a.reduce(function(x,y{ return (x>y)?x:y;})) // largest value
```

`reduceRight()` works just like reduce, but from the largest array index to the lowest.

### indexOf() and lastIndexOf()

returns =1 if not found.  The value is the value to search for, second value is optional and specifies at what index to start the search.

### isArray

[comment]: <> (TODO: Need to play around with this some more.)

`Array.isArray([])` will return true