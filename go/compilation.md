---
layout: page
title: "Compiling a Go Program"
permalink: /go/compiling
---

## Building and running Go binaries

To compile your Go program run use the command `go build my_file.go`.  This will produce a binary executable which you can then run.

To compile and run immediately you can use the command `go run my_file.go`.

## Structure of a Go Program

The first line of a .go file should declare the package the file belongs to using the `package` keyword.  For example `package main`.

The entry point of your program must be a function named `main()` in the `main` package.