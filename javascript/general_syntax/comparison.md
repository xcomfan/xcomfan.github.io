---
layout: page
title: "Comparison Operators"
permalink: /javascript/compare
---

## Strict equality operator (== vs ===)

`===` is the strict equality operator in JavaScript.  `==` is the equality operator.  The difference between the two is `==` will try to convert both types being compared to a common type, `===` does not do the conversion.  If the values being compared with a `===` operator are not the same type they are considered unequal and the strict equality operator will return `false`.

## Inequality and strict inequality operators

```javascript
3 !== 3 // false
3 !== '3' // true
4 !== 3 // true

1 != 2 // true
1 != "1" //false
1 != 1 // false
0 != false // true
```

## Greater and less than operators

The `<`, `<=`, `>`, `>=` operators all do not have a strict counterpart and will try to convert to a common type for the comparison.

