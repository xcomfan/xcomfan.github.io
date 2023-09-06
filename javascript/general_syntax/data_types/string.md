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

## String properties

You can use the `.length` property to get the length of a string `console.log("Some Text".length)`

To check if a string is empty use the expression `if (value === "")`
