---
layout: page
title: "Go Interfaces"
permalink: /go/interfaces
---

## Basic usage

Interface is a type that provides a list of methods but provides no implementation.

```go
package main

import "fmt"

// define interface
type tester interface {
    // list the signature types that make up our interface
    test(int) bool
}

// function takes arg of int and a slice of types that implement tester
func runTests(i int, tests []tester) bool {
    result := true
    for _, test := range tests {
        result = result && test.test(i)
    }
    return result
}

type rangeTest struct {
    min int
    max int
}

// there is no explicit declaration of interface implementation
// as long as your type implements the right methods it meets the interface
// both name and signature need to match or code won't compile.
func (rt rangeTest) test(i int) bool {
    return rt.min <= i && i <= rt.max
}

// define type divTest based on int
type divTest int

func (dt divTest) test(i int) bool {
    return i%int(dt) == 0
}

func main() {
    result := runTests(10, []tester{
        rangeTest{min: 5, max: 20},
        divTest(5),
    })
    fmt.Println(result)
}
```

## Empty interfaces

If you have an interface with no methods at all that is an **empty interface**.  An empty interface is a way to express that object could be anything.

```go
func main() {
    // declare the empty interface
    // any type in go will match it even built in types
    // its a way to say this could be anything like Object type in Java
    var i interface{}
    i = "Hello"
    // assert the type behind i is a sting and assign it to j
    j := i.(string)
    // ok will be assigned to a bool of assertion result
    // if assertion true k will be assigned teh value if it fails
    // k will be assigned the 0 value of that type.
    k, ok := i.(int)
    fmt.Println(j, k, ok)
    // line below will panic if assertion fails
    // use the k, ok version above if doing type assertions
    // m := i.(int)
    // fmt.Println(m)
}
```

## Getting the concrete type behind an interface

```go
package main

import "fmt"

func doStuff(i interface{}) {
    // type is a keyword and by convention the variable on the left hand side
    // of := re-uses the variable on the right hand side in a type assertion
    switch i := i.(type) {
    case int:
        fmt.Println("Double i is", i+i)
    case string:
        fmt.Println("i is", len(i), "characters long")
    default:
        fmt.Println("I don't know what to do with this.")
    }
}

func main() {
    doStuff(10)
    doStuff("Hello")
    doStuff(true)
}
```

## Making a function implement an interfaces

[comment]: <> (TODO: I need to work through this section as I don't quite understand it yet)

You can make a function implement an interface by defining a function type and a method on the function type.

This lets you convert functions with the right functions signature into instances of a type.  Lets you use closure as an instance of an interface.

```go
package main

import "fmt"

type tester interface {
    test(int) bool
}

func runTests(i int, tests []tester) bool {
    result := true
    for _, test := range tests {
        result = result && test.test(i)
    }
    return result
}

// type testerFunc that is of type function that takes in int and returns a bool
type testerFunc func(int) bool

// declare method on testerFunc that takes in an int and returns a bool.
func (tf testerFunc) test(i int) bool {
    // call the underlying function and return its return value.
    // because testerFunc has method that takes an int and returns a bool
    // it effectively implements the tester interface..
    return tf(i)
}

func main() {
    result := runTests(10, []tester{
        // anonymous function that uses type conversion to make it into a testerFunc
        // provides behavior for interface on the fly rather than having to define it before
        testerFunc(func(i int) bool {
            return i%2 == 0
        }),
        testerFunc(func(i int) bool {
            return i < 20
        }),
    })
    fmt.Println(result)
}
```