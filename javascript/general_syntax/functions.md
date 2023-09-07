---
layout: page
title: "JavaScript Functions"
permalink: /javascript/functions
---

## Defining a function

```javascript
function functionName(param1, param2){
  console.log("Hello World");
  return 42;
}
```

## Arrow Functions

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

If your function does not return a value the default return value will be undefined.

## ES6 Enhancements To Functions

ES6 adds ability to set default values for functions.

```javascript
const greeting = (name = "Anonymous") => "Hello " + name;
console.log(greeting("John"));
console.log(greeting());
```

In order to help us create more flexible functions ES6 introduces the `rest` parameter for function parameters.  With the rest parameter, you can create functions that take a variable number of arguments.  These arguments are stored in an array that can be accessed later from inside the function.

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
    return 'Hello! My name is ${this.name}'
  }
}
```

## Global/Local Scope And Functions

Variables declared outside of a function have global scope (can be seen everywhere in your JavaScript code).  Variables created without let or const keyword are automatically created in the global scope.  This can lead to unintended consequences thus you should always declare your variables with `let` or `const`.  Variables declared in a function as well as function parameters have local scope (only visible within the function).  Its possible to have both local and global variables with the same name.  When this happens the local variable takes precedence over the global.

## Arguments Passes By Value and Reference

### Arguments are passed by value

JavaScript function arguments are passed by value not by reference.  The function only gets to know the values, not the arguments locations.  If a function changes an argument's value, it does not change the parameter's original value.

### Objects are passed by reference

In JavaScript, object references are values.  Because of this, objects will behave like they are passed by reference.  If a function changes an object property, it changes the original value.  *Changes to object properties are visible (reflected) outside the function.*

## Returning Boolean Values form Functions

```javascript
function isEqual(a,b){
  return a === b;
}
```
