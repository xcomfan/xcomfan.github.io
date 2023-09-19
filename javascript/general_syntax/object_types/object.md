---
layout: page
title: "Javascript Objects"
permalink: /javascript/objects
---

## Object properties

## General info

[comment]: <> (TODO: Need to break up this section and verify the concepts by running the code yourself.)

Property names can be any string including the empty string.

Objects will inherit the properties (including functions) of its prototype.

In addition to properties each object has three associated *object attributes*

* Prototype reference
* Class string that categorizes the type of object
* extensible flag which in ECMA5 specified if new properties may be added to the object.
  * Use `Object.isExtensible()` to check if an Object is extensible
  * Use `Object.preventExtensions()` to make an object non extensible.  Once an object is extensible there is no way to make it extensible again.  This only effects the object itself.  If new properties are added to the prototype of a non extensible object they will be added to the object as well.
  * `Object.seal()` works like Object.preventExtensions(), but it also makes the own properties of that object non-configurable. This means new properties can't be added and existing properties can't be deleted or configured.  Existing writable properties can still be set.  There is no way to unseal sealed objects.  You can use `Object.isSealed()` to check if an object is sealed.
  * `Object.freeze` In addition to making the object non-extensible and its properties non-configurable it also makes all of the objects own data properties read only.  If the object has accessor properties with setter methods, those are not effected and can still be invoked.  You can use `Object.isFrozen()` to determine if an object is frozen.
  * The `Object.preventExtensions()`, `Object.seal()`, and `Object.freeze()` all return the object they are passed which means you can use them in nested function invocations. For example `let o = Object.seal(Object.create(Object.freeze({x:1},{y: {value: 2, writable: true}})));`

### The global object

To access the global object sue the this keyword from any top level JavaScript code (not in a function for example).

If your code declares a global variable, that variable is a property of the global object.

In client-side JavaScript, the Window object serves as the global object for all JavaScript code contained in the browser windows it represents.  To access the global Window Object has a self referrential property that can be used instead of this to refer to the global object.

### Accessing properties of an object

You access data in objects by **properties**.  You can access data using either dot `.` or bracket `[]` notation.  If the property you are trying to access has a space in the name, you have to use bracket notation.

```javascript
const myObj = {
  prop1: "val1",
  prop2: "val2"
};

const prop1val = myObj.prop1
const prop2val = myObj.prop2

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

### Adding properties to an object

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

### Getting all the keys or values of an object

[comment]: <> (TODO: Need to test this code and make a better example.)

To get all the keys of an object `const keys = Object.keys(object)`

To get all the values of an object `let d = Object.values(countsByDate)`

### Checking if an object has a certain property

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

### Destructuring Objects

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

### ES6 Object Literal Syntactic Sugar

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

### Creating Objects with the class keyword

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

### Using ES6 getters and setters to control access to an object

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

### Freezing an object

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

[comment]: <> (TODO: The notes below are from my older note set.  Need to validate them and merge into content above in a sensible manner.)

##  Object comparison

Objects are not compared by value.  Two objects are not equal if they have the same properties and values.  Similarly.  two arrays are not equal even if they have the same elements.  

For <,>,<=,>= of object comparison is done as follows if you don't provide a comparison method of your own.  

* First either the valueOf or the toString() method is used to convert the object down to a primative.  ValueOf is tried first and toString as a fallback.
* If after the conversion to primative both are string they are compared alphabetically based on the numerical code of the 16-bit unicode.
* If after the primative conversion at leat one operator is not a string both operands are converted to numbers and compared numerically.
* The **instanceof** operator does not refer to the constructor.  It checks the prototype property of the class and returns true if it matches.

Two object values are the same if and only if they refer to the same underlying object.  

Assigning an object or array to a variable simply assigns the reference it does not create a new copy.  If you want to make a new copy you have to explicitly copy the properties of the object or the elements of an array.

To enable instances of your class to be tested for equality implement the equals() method.

## Object Properties

Accessor properties are defined as one or two functions whose name is the same as the property name.  and with the function keyword replaced with get or set.  **Note** no colon is used to separate the name of the function from the property.

```javascript
var p = {
  x: 1.0,
  y: 10,

  //r is a read-write accessor property with getter and setter 
  // Dont forget to put a comma after accessor methods.
  get r() { return Math.sqrt(this.x*this.x + this.y*this.y)},
  set r(newValue) {...}

}...
```

Note how this is used in getters and setters.  Javascript invokes these functions as methods of the object on which they are defined.

A good use of getters and setters is to do sanity checking on values that someone will try to set.

### Rules of properties

If an object is not extensible, you can edit it's existing own properties, but you cannot add new properties to it.

If a property is not configurable, you cannot change its configurable or enumerable attributes.

If an accessor property is not configurable, you cannot change its getter or setter method, and you cannot change it to a data property

If a data property is not configurable, you cannot change it to an accessor property.

If a data property is not configurable, you cannot change its writable attriubte from false to true, but you can change it from true to false.

if a data property is not configurable and not writable you cannot change it's value.  You can change the value of a property that is configurable but nonwritable, however.

## Prototype chain

To determine whether one object is the prototype of or is part of the prototype chain of another object use the `isPrototypeOf()` method.

```javascript
let p = {x:1};
let o = Object.create(p)
p.isPrototypeOf(o) // true o inherits from p
Object.prototype.isPrototypeOf(0) // true p inherits from Object.prototype
```

## Standard Conversion Methods

You don't need to implement these for every class you write just a list for reference of standard object methods.

* toString()
* valueOf()
* toJSON()

## Object Serialization

Use JSON.stringify() and JSON.parse() to serialize and restore JavaScript objects.

Nan and Infinity and -Infinity are serialized to null

Data objects are serialzed to ISO formatted date strings.

JSON.parse() leaves these in string form and does not restore the original Data object.

RegExp, and Error objects and undefined value cannot be serialized or restored.

JSON.stringify() serialized only the enumerable own properties of an object.

If a property cannot be serialized that property is simply omitted from the stringified output.

Both JSON.strigify() and JSON.parse() have optional addition arguments that augment their behaviour (such as passing a list of attriubtes to stringify) see full docs for more.
