---
layout: page
title: "Variables in Go"
permalink: /go/variables
---

## Declaring a variable

Variables are declared with the `var` keyword and can be declared either in a function or a package.

The format of declaration is the keyword `var` followed by the variable name followed by the type and optionally assignment.

```go
var i int = 10
```

If you are declaring multiple variables of the same type you can list the type at the end.

```go
// with type 
var i, j int = 1, 2
// with inferred type
var c, python, java bool = true, false, "no!"
```

Variables can be declared without initializing them.

```go
var c, python, java 
```

## Short variable declaration


If you are declaring a variable inside a function the `:=` shorthand can be used in place of a var declaration with implicit type.

```myString := "Hello World!```

## Exported names

[comment]: <> (TODO: I don't yet have good notes on packaging.  Will move this there when I do.)

In Go, a name is exported if it begins with a capital letter.  When a package is imported you can only access variables from the package which are exported.

## Variable zero values

In Go variables that are declared but have no value assigned are assigned the **zero value** of that type.  For example numeric types will be default assigned the zero value of `0`.

## Constants

Constants are declared like variables, but with the `const` keyword

Constants can be character, string, boolean, or numeric types.

Constants cannot be declared using the `:=` syntax
