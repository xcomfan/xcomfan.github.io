The syntax for a ternary operator in JavaScript is a ? b : c, where a is the condition, b it the code tor un when the condition returns true, and c is the code to run when the condition returns false.

```javascript
function checkEqual(a,b){
  return a === b ? "Equal" : "Not Equal";
}
```

You can also nest ternary operators

```javascript
function findGreaterOrEqual(a,b){
  return (a === b) ? "a and b are equal"
    : (a > b) ? "a is greater"
    : "b is greater";
}
```

Its considered best practice to format conditional operators such that each condition is on a separate line as in the example above.
