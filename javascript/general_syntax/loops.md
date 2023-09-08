---
layout: page
title: "JavaScript Loops"
permalink: /javascript/loops
---

## while loop

Loops though a block of code as long as a specified condition is true.

```javascript
let count = 0;
while (count < 10){
    console.log(count);
    count++;
}
```

## do/while loop

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

## for loop

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

## for in

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

## for of

The JavaScript `for of` statement loops through the values of an iterable object.  It lets you loop over iterable data structures such as Arrays, Strings, Maps, NodeLists and more.

## Looping over an array

```javascript
const cars = ["BMW", "Volvo", "Mini"];

let text = "";
for (let x of cars) {
  text += x;
}
```

## Looping over a string

```javascript
let language = "JavaScript";

let text = "";
for (let x of language) {
text += x;
}
```

## Array.forEach()

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
