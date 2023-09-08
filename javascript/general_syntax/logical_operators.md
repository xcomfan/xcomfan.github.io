---
layout: page
title: "JavaScript Logical Operators"
permalink: /javascript/logical_operators
---

## Truthy and falsy in JavaScript

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

## Logical AND

Logical AND operator is written as `&&` in JavaScript.  For example `a > 0 && b > 0`

### Logical AND assignment

The logical AND assignment `&&=` operator only evaluates the right operand and assigns to the left if the left operand is truthy.

```javascript
let a = 1;
let b = 0;

a &&= 2;
console.log(a); // expected output 2

b &&= 2;
console.log(b); // expected output 0
```

## Logical OR

Logical OR operator is written as `||` in JavaScript.  For example `a > 0 || b > 0`

### Logical OR assignment

The logical OR assignment `||=` operator only evaluates the right operand and assigns to the if the left operand is falsy

```javascript
const a = { duration: 50, title ''};

a.duration ||= 10;
console.log(a.duration); // expected output 50

a.title ||= 'title is empty.';
console.log(a.title); // expected output "title is empty"
```

## Logical NOT

The logical NOT is operator is written as `!` in JavaScript.  For example `!(a > 0 || b > 0)`
