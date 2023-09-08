

## Types that can contain values

[String]({% link javascript/general_syntax/data_types/string.md %})

[Number]({% link javascript/general_syntax/data_types/number.md %})

[Boolean]({% link javascript/general_syntax/data_types/boolean.md %})

## Object types

[Object]({% link javascript/general_syntax/object_types/object.md %})

[Array]({% link javascript/general_syntax/object_types/array.md %})

[Date]({% link javascript/general_syntax/object_types/date.md %})

## Types that cannot contain values

[Null]({% link javascript/general_syntax/data_types/undefined_and_null.md %})

[Undefined]({% link javascript/general_syntax/data_types/undefined_and_null.md %})

## Type Operator

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