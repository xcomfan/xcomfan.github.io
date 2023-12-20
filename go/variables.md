---
layout: page
title: "Variables in Go"
permalink: /go/variables
---

## Declaring a variable

Variables are declared with the `var` keyword and can be declared either in a function or a package.

The format of declaration is that keyword `var` followed by the variable name followed by the type and optionally assignment.

```go
var i int = 10
```

## Declaring a variable with type inference

If you are declaring and assigning a variable in one line Go can frequently guess the type of variable being declared.  While the expression `var i = 10` is valid in Go there is a shorthand that used the `:=` operator.

To declare a variable when type can be inferred you can use the expression `i := 10`.  ***Note:*** this is only allowed for function variables.  Package variables must use the `var` keyword for declaration.

You cannot use `:=` operator on a variable that has already been assigned as the `:=` operator is assignment not declaration.

## Variable zero values

In Go variables that are declared but have no value assigned are assigned the **zero value** of that type.  For example numeric types will be default assigned the zero value of `0`.