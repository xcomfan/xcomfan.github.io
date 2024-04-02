---
layout: page
title: "Go Packages And Imports"
permalink: /go/packages
---

## Basic Info

Every Go program is made up of packages.  The package `main` is the entry point to your Go program. 

A good place to look for go packages is [pkg.go.dev](https://pkg.go.dev)

## Using packages

Go packages fall into 3 categories

* Packages you create

* Packages other people create

* Packages in the standard library

You have to import entire packages.  You cannot import a single item from a package like you can in Python.

You import packages into your code to use with the `import` statement.

```go
import (
    "fmt"
    "math"
)
```

## Creating and using your own packages

Your own package should live in its own separate directory.

For all packages other than `main` the package name at the top of the go file should match the directory name.  By convention the package name is the same as the last element of the import path.  For example, "math/rand" package is made up of files that begin with the statement `package rand`.

### Variable scope

You can declare variables at the package level.  These variables cannot be assigned with the `:=` operator, and need to be declared outside a function.  If declared this way they are accessible in the entire package.  Package level variables can be superseded by local ones (example below)

```go
package main

import (
    "fmt"
)

var val = "Package Level Value"

func main() {
    val := "Local Scop"
    fmt.Printf("Type of val is %T value is %s\n", val, val)
    // calling the function lets as access global version of val
    printVal()
}

func printVal() {
    fmt.Printf("Global val is: %s\n", val)
}
```

### Variable visibility

In Go there are 2 levels of visibility.  **package visible** and **public** and these apply to variables, functions and data types.  *if you want to make something public you capitalize the first letter in the variable name*

## Importing

Go allows indirect imports but its considered bad practice.  Try to use full path in your imports.

It is illegal to import a package but not use anything from it.  This will give you a syntax error during compilation.  Cyclical dependencies are also not allowed.

## Third party packages

[comment]: <> (TODO: Need to flesh out this section a bit.)

To install a package into your local Go path use the command `go get package_name`.  Once you do that you can use the package in your code.

You install dep by running an install script and then you can run dep init command to create Gopkg.toml files and Gopkg.lock files.  Use `dep ensure` to control what packages get installed.  More info at <https://golang.github.io/dep/>

## Packaging

You should have a comment before all exported functions (ones that start with a capital)

[comment]: <> (TODO: Module section should be expanded and moved to it's own seciton but its OK here for now whie its content is kind of slim.)

## Modules

A module is a collection of packages.

You can tell that something is a module because there will be a `go.mod` file that exists at the root of the project.

[comment]: <> (TODO: Not sure if the below is true but that is my current understanding.)

Your code will need to be a module if you want to use third party packages.  To make your code a module use the command `go mod init my_module_name`.  This will generate the `go.mod` file.  Once you have the `go.mod` file generated you are able to run the command `got get package_path` to download packages into your cache and `go mod tidy` command to add/remove packages from cache based on your code imports.