---
layout: page
title: "General Syntax"
permalink: /javascript/general_syntax
---

[General Information]({% link javascript/general_syntax/general_information.md %})

[Code Comments]({% link javascript/general_syntax/comments.md %})

[Variables]({% link javascript/general_syntax/variables.md %})

[Data Types]({% link javascript/general_syntax/data_types/data_types.md %})

[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})
[TEMP]({% link javascript/general_syntax/comments.md %})


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

### Null

In JavaScript null is "nothing".  It is supposed to be something that doesn't exist.  Unfortunately, in JavaScript, the data type of null is an Object.  

Its should be considered a bug that `typeof` for null is an Object.

You can empty an object by setting it to null

```javascript
var person = {firstName: "John", lastName: "Doe", age: 50};
person = null; // Now the value is null but type is an object.
```

The difference between undefined and null is that null is an Object and `typeof` undefined is undefined.  In other words `null` is for objects and `undefined` is for variables, properties and methods.

## Functions

### Function Definition

```javascript
function functionName(param1, param2){
  console.log("Hello World");
  return 42;
}
```

If your function does not return a value the default return value will be undefined.

### Global/Local Scope and Functions

Variables declared outside of a function have global scope (can be seen everywhere in your JavaScript code).  Variables created without let or const keyword are automatically created in the global scope.  This can lead to unintended consequences thus you should always declare your variables with `let` or `const`.  Variables declared in a function as well as function parameters have local scope (only visible within the function).  Its possible to have both local and global variables with the same name.  When this happens the local variable takes precedence over the global.

### Returning Boolean Values form Functions

```javascript
function isEqual(a,b){
  return a === b;
}
```

### Arrow Functions

In JavaScript we don't need to name our functions; especially when passing a function as an argument to another function.  Instead we create inline functions.

We don't need to name these functions because we do not reuse them anywhere else.  To achieve this we use the following syntax.

```javascript
const myFunc = function(){
  const myVar = "value";
  return myVar;
}

// ES6 provides the syntactic sugar of arrow functions
const myFunc = () => {
  const myVar = "value"
  return myVar;
}
```

When there is no function body and just a return value, arrow function syntax allows you to omit the keyword return as well as the brackets surrounding the code.  This helps simplify smaller functions into one line statements.

`const myFunc = () => "value";`

Just like a regular function, you can pass arguments into an arrow function.

`const doubler = (item) => item * 2;`

If the arrow function has a single parameter you can omit the parentheses around the arguments as well.

`const doubler = item => item * 2`;

It is possible to pass more than one argument into an arrow function.

```javascript
const multiplier = (item, multi) => item * multi;
multiplier(4,2);
```

***Note:*** An arrow function does not bind `this` to the parent context.  The `this` inside an arrow function will not be the `this` in the outside scope.

### ES6 Enhancements to Functions

ES6 adds ability to set default values for functions.

```javascript
const greeting = (name = "Anonymous") => "Hello " + name;
console.log(greeting("John"));
console.log(greeting());
```

In order to help us create a more flexible functions ES6 introduces the `rest` parameter for function parameters.  With the rest parameter, you can create functions that take a variable number of arguments.  These arguments are stored in an array that can be accessed later from inside the function.

```javascript
function howMany(...args){
  return "You have passed " + args.length + " arguments.";
}

console.log(howMany(0,1,2));
console.log(howMany("string", null, [1,2,3], {}));
```

When defining functions within objects in ES5, we have to use the keyword `function`.  ES6 gives us a shorthand.

```javascript
// ES5 Way
const person = {
  name: "Taylor",
  sayHello: function() {
    return `Hello! My name is ${this.name}`;
  }
}
// ES6 Way
const person = {
  name: "Taylor",
  sayHello() {
    return 'Hello! My name is ${this.name}`
  }
}
```

### Arguments are passed by value

JavaScript function arguments are passed by value not by reference.  The function only gets to know the values, not the arguments locations.  If a function changes an argument's value, it does not change the parameter's original value.

#### Objects are passed by reference.

In JavaScript, objct references are values.  Because of this, objects will behave like they are passed by reference.  If a function changes an object property, it changes the original value.  *Changes to object properties are visible (reflected) outside the function.*

## Data Structures

### Arrays

#### Adding and removing data to an array

To append data to the end of an array use the `push()` function.  To remove data from the end (and get the value) use the `pop()` function.  To remove value from front of array and get its value use the `shift()` function.  To insert a value at the beginning of an array use the `unshift()` function.

```javascript
const arr1 = [1,2,3];
arr1.push(4)

let last = arr1.pop()
let first = arr1.shift()

arr1.unshift(last);
```

To get the length of an array use the length property. `len = myArr.length`

#### Using the spread operator

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

#### Destructuring Arrays

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

### Objects

#### Object properties

You access data in objects by **properties**.  You can access data using either dot `.` or bracket `[]` notation.  If the property you are trying to access has a space in the name, you have to use bracket notation.

```javascript
const myObj = {
  prop1: "val1",
  prop2: 'val2"
};

const prop1val = myObj.prop1
const prop1val = myObj.prop2

const myObj2 = {
  "Space Name": "Kirk",
  "More Space": "Spock",
  "NoSpace": "USS Enterprise"
};

let captain = myObj2["Space Name"];
let otherGuy = myObj2["More Space"];
let ship = myObj2["NoSpace"];
```

you can use variables to access object properties

```javascript
const dogs = {
  Fido: "Mutt",
  Hunter: "Doberman",
  Snoopie: "Beagle"
};

const myDog = "Hunter";
const myBreed = dogs[myDog];
console.log(myBreed);
```

Adding properties to an object looks just like assignment.

```javascript
const ourDog = {
  "name": "Luis",
  "legs": 4,
  "tails": 1,
  "friends": ["everything!"]
};

ourDog.bark = "bow-wow" // ourDob["bark"] = "bow-wow"; is also valid
```

To delete a property from an Object use the delete operator

```javascript
const ourDog = {
  "name": "Luis",
  "legs": 4,
  "tails": 1,
  "friends": ["everything!"],
  "bark": "bow-wow"
};

delete ourDog.bark
```

To verify if an object has a certain property use the `hasOwnProperty()` function.

```javascript
const myObj = {
  top: "hat",
  bottom: "pants",
};

myObj.hasOwnProperty("top"); // will return true
myObj.hasOwnProperty("middle"); // will return false
```

```javascript
else if (prop === 'tracks' && records[id].hasOwnProperty('tracks') === false){
  records[id][prop] = value;
}
```

[comment]: <> (TODO: Need to test this code and make a better example.)

To get all the keys of an object `const keys = Object.keys(object)`

To get all the values of an object `let d = Object.values(countsByDate)`

#### Destructuring Objects

With destructuring of an object you can assign values directly from an object.

```javascript
const user = {name: 'John Joe', age: 34};
const name = user.name;
const age = user.age;

// with destructuring you can just write

const {name, age } = user

// If you want to be able to name the variables use the syntax

const { name: userName, age: userAge} = user
```

You can destructure an object when passing it to a function to extract the exact properties you want as in the example below.

```javascript
const stats = {
  max: 56.78,
  standard_deviation: 4.34,
  median: 34.54,
  mode: 23.87,
  min: -0.75
  average: 35.85
};

const half = ({max, min} = stats) => (max + min) / 2.0;
```

#### ES6 Object Literal Syntactic Sugar

ES6 provides syntactic sugar to eliminate the redundancy of having to write x:x when defining an object literal.  You can just write it once for the property.  Fore example

```javascript
// ES5 Way
const getMousePosition = (x,y) => ({
  x: x,
  y: y
});

//ES6 Way
const getMousePosition = (x,y) => ({x,y});
```

#### Creating Objects with the class keyword

ES6 Provides a new syntax to crate objects, using the `class` keyword.  This is just syntax, not a full class based implementation of an object oriented paradigm unlike Java or Python.  

In ES5 we usually define a constructor function and use the new keyword to instantiate an object.

```javascript
var SpaceShuttle = function(targetPlanet){
  this.targetPlanet = targetPlanet;
}

var zeus = new SpaceShuttle("Jupiter");
```

In ES6 the class syntax simply replaces the constructor function creation.

```javascript
class SpaceShuttle{
  constructor(targetPlanet){
    this.targetPlanet = targetPlanet;
  }
}

const zeus = new SpaceShuttle('Jupiter');
```

#### Using ES6 getters and setters to control access to an object

Getter functions are meant to simply return (get) the value of an object's variable to the user without the user directly accessing the private variable.  Setter functions are meant to modify (set) the value of an objects private variable based on the value passed into the setter function.  This change could involve calculations, or even overwriting the previous value completely.  Getters and setters are important because they hide implementation details.  

***Note*** It is convention to precede a private variable with an _.

```javascript
class Book {
 constructor(author){
  this._author = author;
 } 
 // getter
 get writer() {
  return this._author;
 }
 // setter
 set writer(updatedAuthor){
  this._author = updatedAuthor;
 }
}

const novel = new Book('anonymous');
console.log(novel.writer)
novel.writer = 'newAuthor';
console.log(novel.writer);
```

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

## Type Operators

`typeof` - Returns the type of a variable

```javascript
typeof "John" // Returns "string"
typeof 3.14 // Returns "number"
typeof Nan // Returns "number"
typeof false // Returns "boolean"
typeof [1,2,3,4] // Returns "object"
typeof {name: "John", age: 34} // Returns "object"
typeof function () {} // Returns "function"
typeof myThing // Returns "undefined"
typeof null // Returns "object"
```

`instanceof` -  Returns true if an instance of an object type.

```javascript
const cars = ["Pontiac", "BMW", "Nissan", "Ford", "Lexus"];

(cars instanceof Array) // Returns true
(cars instanceof Object) // Returns true
(cars instanceof String) // Returns false
(cars instanceof Number) // Returns false
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

## ES6 xxports/imports for sharing code blocks

[comment]: <> (TODO: Need to run though this section and make corrections if needed based on testing it myself.)

### Exporting

In order to use a function defined in one file to be used in another you need to export it.  To export a single function...

```javascript
export const add = (x, y) => {
  return x + y;
}
```

If you need to export multiple functions...

```javascript
const add = (x,y) => {
  return x + y;
}

const subtract = (x,y) => {
  return x - y;
}

export{ add, subtract };
```

You can create an export fallback with export default.  This is usually done if one value is being exported from a file or to create a fallback value for file or module.  Your are not limited to functions with exporting.  You an export variables declared via `var`, `let` or `const` as well.

```javascript
// default export function
export default function(x, y){
  return x + y
}
```

When using export default on the import side you just specify the file not what you are importing.  In the example below you are naming the function being imported add.

```javascript
import add from "./math_functions.js"
```

### Importing

You can import specific exports.

```javascript
import { add } from './math_functions.js';
import { add, subtract } from './math_functions.js';
```

You can use `import * as` syntax to import all contents of a file.

```javascript
import * as myMathModule from "./math_functions.js";

myMathModule.add(2,3);
myMathModule.subtract(5,3);
```

## Promises

[comment]: <> (TODO: Need to reivew update work though examples and test this section..)

A promise in JavaScript is exactly what it sounds like.  You use it to make a promise to do something, usually asynchronously.  Whe the task completes you either fulfill the promise of fail to do so.  

A promise is a constructor function, so you need to use the `new` keyword to create one.  It takes a function as its argument with two parameters - resolve and reject.  These are the methods to determine the outcome of the promise.  

A promise has 3 states:

* pending
* fulfilled
* rejected

```javascript
const myPromise = new Promise((resolve, reject) => {
  if (condition here){
    resolve("Promise was fulfilled");
  } else {
    reject("Promise was rejected");
  }
});
```

The example above uses strings for the arguments of these functions, but it can really be anything.  Often, it might be an object, that you would use data from, to put on your website or elsewhere.

Promises are most useful when you have a process that takes an an unknown amount of time in your code.  (i.e., something asynchronous) often a server request.

When you make a server request, it takes some amount of time, and after it completes you usually want to do something with the response from the server.  This can be achieved using the `then` method.  The then method is executed immediately when your promise is fulfilled with the `resolve` method.

```javascript
myPromise.then(result => {

});
```

`catch` is the method used when your promise has been rejected.  It is executed immediately after a promise's reject method is called.

```javascript
myPromise.catch(error => {

});
```

## Exception handling

### throw

`throw` expression

The expression above can evaluate to any time.  You can throw a number that represents an error code or a human readable string.

When an expression is thrown in JavaScript the interpreter stops normal program execution and jumps to the nearest exception handler.

### try catch finally

```javascript
try {
  // some coe that will run unless there is a problem.
}
catch(e){
  // Statements in this block are executed if and only if the try block throws an exception.
  // e in this example refers to the error thrown.
}
finally{
  // This block is always executed.
}
```

## Useful Tools

### Generating a Random Number

JavaScript has a Math.random() function that generates a random decimal number between 0 (inclusive) and 1 (exclusive) thus Math.random() can return a 0 but never return a 1.

To generate a random number between 0 and 19 

```javascript
Math.floor(Math.random() * 20); // Note the 20 not 19 since Math.random() will never return 1.
```

### Working with Dates

```javascript
var dt = new Date();

dt.getFullYear() + "/" (dt.getMonth() + 1) "/" dt.getDate();

//Function for getting the Sunday(or another day with a teak start date)
export function getStartOfWeekDate(date){
  d = new Date(date);
  let date = d.getDay(),
  // diff = d.getDate() - day + (day == 0 ? -6:1); //adjust when day is Sunday
  diff = d.getDate() - day;
  return new Date(d.setDate(diff));
}
```
