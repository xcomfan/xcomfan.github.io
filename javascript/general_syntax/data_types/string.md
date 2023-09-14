---
layout: page
title: "JavaScript String"
permalink: /javascript/string
---

## Declaring strings

Strings are defined using either single or double quotes.

`const myStr = 'This is my string'` or `const myStr = "This is my string"`

`"` and `'` characters in a string can be escaped with a `\`

## Strings are immutable

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

A call to `string.toUpperCase()` and `string.replace()` will return a new string not change the existing one in place.

## Concatenating Strings

Strings can be concatenated with the `+` and `+=` operators.

```javascript
const ourStr = "I come first. " + "I come second."

const anAdjective = "awesome!";
let ourStr = "Pizza is";
outStr += anAdjective;
```

You can also concatenate a string as a mix of constants and variables.  For example `const ourStr = "Hello" + ourName + ", how are you?"`

## ES6 template literals

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

## String Properties

You can use the `.length` property to get the length of a string `console.log("Some Text".length)`

To check if a string is empty use the expression `if (value === "")`

## Some Useful String Methods

```javascript
let s = "hello world";
s.charAt(0) // "h": the first character
s.charAt(s.length-1)  // => "d" the last character
s.substring(1,4)      // => "ell": the 2nd 3rd and 4th characters
s.slice(1,4)          // => "ell": same thing as above
s.slice(-3).          // => "rld": last 3 characters
s.indexOf("l")        // => 2: position of first letter l
s.lastIndexOf("l")    // => 10: position of last letter l
s.indexOf("l",3)      // => 3: position of first "l" at or after 3
s.split(",")          // => ["hello","world"] split into substrings
s.replace("h","H")    // => "Hello world": replaces all instances
s.toUpperCase()       // => "HELLO WORLD"
```
