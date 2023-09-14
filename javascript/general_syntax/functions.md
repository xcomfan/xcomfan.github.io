---
layout: page
title: "JavaScript Functions"
permalink: /javascript/functions
---

## Functions are an object

Functions are a special kind of object in JavaScript and can be treated like a regular object.

A function can be a property of another object in which case it becomes a method.  When you call a method you implicitly pass the `this` value to refer to the object the function belongs to.

## Defining a function

```javascript
function functionName(param1, param2){
  console.log("Hello World");
  return 42;
}
```

Function declaration statements are hoisted to the top of the enclosing script or the enclosing function.  This is not true for functions defined as expressions as you won't be able to refer to a function defined as an expression.

Function definitions cannot appear inside of loops conditionals or try/catch/finally and with statements.

### Function arguments and parameters

JavaScript does not check the type or number of arguments passed to a function.  When a function is invoked with fewer arguments than parameters the unset parameters are defaulted to undefined.

You can give give a default value to an argument in shorthand using `a = a || []`

When a function is invoked with more argument values than there are parameter names, there is no way to directly refer to the unnamed values. The `argument` object is a work around.  It is an array like object that allows the argument values passed to the function to be retrieved by number rather than by name.

[comment]: <> (TODO: This example is crap fix it.)

```javascript
function f(x, y, z){
  // First verify the correct number of arguments was passed
  if (arguments.length != 3){
    throw new Error("function f called with " + arguments.length + " but expected 3");
    // now do the actual function
  }
}
```

A useful technique is to have your function accept an object that way you can use name value pairs to access the argument.

Functions can be passed as arguments to other functions.  For example you can have a function that acts on 2 arguments and pass that function along with two operands to the other function.

### Function properties

When a function needs a "static" variable whose value persists across invocations, it is often convenient to use a property fo the function instead of global namespace.

```javascript
// Initialize the counter property of the function object
// Function declarations are hoisted so we can do this assignment before declaration.
uniqueInteger.counter = 0
function uniqueInteger(){
  return uniqueInteger.counter ++; //
}
```

### Return statement

The `return` statement causes a function to stop.  If a value is not returned it defaults to undefined.

## Arrow functions

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

### Nested functions

```javascript
function hypotenuse(a,b){
  function square(x) { return x*x; }
  return Math.sqrt(square(a) + square(b));
}
```

Functions can access the parameters of the function they are nested in.  In the above example the inner function square can read and write the parameters a and b defined by the outer functions hypotenuse.

Unlike variables the `this` value does not have a scope and nested functions do not inherit the `this` value of their caller.  If you want to access the this value of the caller you need to store it in an argument variable.

## Functions as namespaces

Since functions act as their own namespaces, it is sometimes useful to declare a function to act as an isolated namespace.  Below is an example of defining and calling a function without naming it.  This is an un-named expression.  The open perenthesis before `function` is required because without it JavaScript interpreter will try to parse the word function as a declaration.

```javascript
(function() {
  // Module code goes here.
}());
```

## Closure

JavaScript uses lexical scoping.  This means that functions are executed using the variable scope that was in effect when they were defined, not the variable scope that is in effect when they are invoked.  This combination of a function and scope in which the function's variable are resolved is called a **closure**.

[comment]: <> (TODO: This example is crap fix it.)

```javascript
var scope = "global scope"; // A global variable

function checkscope(){
  var scope = "local scope"; // A local variable
  function f() { return scope; } // Return the value in scope here
  return f;
}

checkscope()()  // This will return local scope as that was the value for scope when function for scope when function was defined.

```

This is the unique intetger example from before but using closures.

```javascript
let uniqueInteger = (function(){
  // define and invoke
  var counter = 0;  // private state of the function that follows below.
  return function(){return counter++}
}())
```

The first line is defining and invoking a function as hinted by the open parenthesis. The nested function has access to the variables in scope and can use the counter variable.  Once the outer function returns no other code can use the counter variable.

It is possible for two or more nested functions to be defined within the same outer function and share the same scope chain.

```javascript
function counter(){
  let n = 0;
  return {
    count: function(){return n++;},
    reset: function(){n = 0;}
  }
}

let c = counter(), d = counter();
c.count()
d.count()
c.reset()
c.count()
c.count()

```

## Built in function methods

### Bind method

The primary purpose of `bind()` is to bind a function to an object.  When you invoke `bind()` on function f and pass an object o, the method returns a new function.  Invoking the new function invokes the original function f as a method of o.  Any argument you pass to the new function are passed to the original function.

```javascript
function f(y) { return this.x + y; } // This function needs to be bound
let o = { x: 1 };  // An object we will bind to.
let g = f.bind(o); // Calling g(x) invokes o.f(x)
g(2) // => 3
```

### toString method

This will usually just give a string with the code of the function.

[comment]: <> (TODO: I need to test this out.)


### Function constructor

The `Function()` constructor allows JavaScript functions to be dynamically created and compiled at runtime.  The `Function()` constructor creates a new function object each time it is called.  if it appears in a loop or function that is often called this can be inefficient.  Nested functions and function definition expressions are not recompiled each time they are encountered.  Functions created from `Function()` constructor do not use lexical scoping.  They are created as top level objects.

[comment]: <> (TODO: Need to find an example of this in action.)

## Returning Boolean Values form Functions

```javascript
function isEqual(a,b){
  return a === b;
}
```
