

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

## Type Conversions

[comment]: <> (TODO: Need to research the ??? sections below as they were not in my notes.  Also what happens if you convert multi element number array to a string and interger?)

| Value | Converted to String | Converted to Number | Converted to Boolean | Converted to Object |
| ---- | ------------------- | ------------------- | -------------------- | ------------------- |
| undefined | "undefined" | NaN | false | *throws TypeError* |
| null | "null" | 0 | false | *throws TypeError* |
| true | "true" | 1 | | new Boolean(true) |
| false | "false" | 0 | | new Boolean(false) |
| "" (empty string) | | 0 | false | new String("") |
| "1.2" (nonempty numeric) | | NaN | true | new String("1.2") | 
| "one" (nonempty, non-numeric) | | NaN | true | new String("one") |
| 0 | "0" | | false | new Number(0)
| -0 | "0" | | false | new Number(-1)
| NaN | "NaN" | | false | new Number(NaN)
| Infinity | "Infinity" | | true | new Number(Infinity)
| -Infinity | "-Infinity" | | true | new Number(-Infinity)
| 1 (finite, non-zero) | "1" | | true | new Number(1)
| {} (any object) | ??? | ??? | true | |
| [] (empty array) | "" | 0 | true | |
| [9] (1 numeric elt) | "9" | 9 | true | |
| ['a'] (any other array) | *use join() method" | NaN | true | |
| function(){} (any function) | ??? | NaN | true | |

### Explicitly Converting Types

If you want to explicitly convert to a type use the `Boolean()`, `Number()`, `String()` or `Object()` functions.

```javascript
Number(3) // 3
String(false) // "false"
Boolean([]) // true
Object(3) // new Number(3)
```
#### Object to Primative Conversions

##### Object to Boolean

All objects will convert to the boolean value of `true`

##### Object to String

All bojects inherit a `toString()` and `valueOf()` method.

To convert to string the `toString()` method is called.  If that returns a primitive, that primitive is converted to a string.

If the object has no `toString()` method, the `valueOf()` method is called and coverts the primitive from that result to a string.

If both of these options do not work JavaScript will throw a TypeError.
