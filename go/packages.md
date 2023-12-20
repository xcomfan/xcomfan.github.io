---
layout: page
title: "Go Packages And Imports"
permalink: /go/packages
---

## Using packages

Go packages fall into 3 categories

* Packages you create

* Packages other people create

* Packages in the standard library

You have to import entire packages.  You cannot import a single item from a package like you can in Python.

## Creating and using your own packages

Your own package should live in its own separate directory.

For all packages other than `main` the package name at the top of the go file should match the directory name.

### Variable scope

You can declare variables at the package level.  These variables cannot be assigned with he `:=` operator, and need to be declared outside a function.  If declared this way they are accessible in the entire package.

### Variable visibility

In Go there are 2 levels of visibility.  **package visible** and **public** and these apply to variables, functions and data types.  ***if you want to make something public you capitalize the first letter in the variable name***

## Importing

Go allows indirect imports but its considered bad practice.  Try to use full path in your imports.

It is illegal to import a package but not use anything from it.  This will give you a syntax error during compilation.  Cyclical dependencies are also not allowed.

## Third party packages

[comment]: <> (TODO: Need to flesh out this section a bit.)

To install a package into your local Go path use the command `go get package_name`.  Once you do that you can use the package in your code.

You install dep by running an install script and then you can run dep init command to create Gopkg.toml files and Gopkg.lock files.  Use `dep ensure` to control what packages get installed.  More info at <https://golang.github.io/dep/>