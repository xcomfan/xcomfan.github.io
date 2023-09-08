---
layout: page
title: "Javascript Objects"
permalink: /javascript/objects
---

## Object properties

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
