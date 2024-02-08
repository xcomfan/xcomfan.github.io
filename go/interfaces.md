---
layout: page
title: "Go Interfaces"
permalink: /go/interfaces
---

## Interface basics

An **interface type** is defined as a set of method signatures but no implementation.  A value of interface type can hold any value that implements those methods.

Interfaces are implemented implicitly. A type implements an interface by implementing its methods.  There is no explicit declaration of intent (no implements keyword or anything like that).  Implicit interfaces decouple the definition of an interface from its implementation, which could then appear in any package without prearrangement.

A good way to think about interfaces.  Think of the shapes circle, triangle and square. If we use interfaces to give each an area method and call the interface Shape, we can now create a slice of shapes.  Even though they are different structs because they implement the same interface we can treat them as the same type and stick them into a slice.  More concrete example of this in Go IO reader is an interface and using this same interface you can work with files, sockets, compressed files etc.

```go
package main

import (
    "fmt"
    "math"
)

type Abser interface {
    Abs() float64
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

type Vertex struct {
    X, Y float64
}

func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    var a Abser
    f := MyFloat(-math.Sqrt2)
    v := Vertex{3, 4}

    a = f  // a MyFloat implements Abser
    a = &v // a *Vertex implements Abser

    // In the following line, v is a Vertex (not *Vertex)
    // and does NOT implement Abser.
    a = v

    fmt.Println(a.Abs())
}
```

## Interface values

Under the hood, interface values can be thought of as a tuple of a value and a concrete type `(value, type)`.  An interface value holds a value of a specific underlying concrete type.  Calling a method on an interface value executes the method of the same name on its underlying type.

```go
package main

import (
    "fmt"
    "math"
)

type MyInterface interface {
    MyInterfaceMethod()
}

type MyType struct {
    S string
}

func (t *MyType) MyInterfaceMethod() {
    fmt.Println(t.S)
}

type MyFloat float64

func (f MyFloat) MyInterfaceMethod() {
    fmt.Println(f)
}

func main() {
    var i MyInterface

    i = &MyType{"Hello"}
    describe(i)
    i.MyInterfaceMethod()

    i = MyFloat(math.Pi)
    describe(i)
    i.MyInterfaceMethod()
}

func describe(i MyInterface) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

### Interface values with nil underlying values

If the concrete value inside the interface itself is `nil`, the method will be called with a `nil` receiver.  While in most languages this would trigger a null pointer exception in Go its common to write methods that gracefully handle being called with a `nil` receiver (as with the method `M` in example below)

***NOTE:*** An interface value that holds a nil concrete value is itself non-nil.

```go
package main

import "fmt"

type I interface {
    M()
}

type T struct {
    S string
}

func (t *T) M() {
    if t == nil {
        fmt.Println("<nil>")
        return
    }
    fmt.Println(t.S)
}

func main() {
    var i I

    var t *T
    i = t
    describe(i)
    i.M()

    i = &T{"hello"}
    describe(i)
    i.M()
}

func describe(i I) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

### Nil interface values

A `nil` interface value holds neither value nor concrete type.  Calling a method on a nil interface is a run-time error because there is no type inside the interface tuple to indicate which *concrete* method to call.

*Example below throws a runtime error*.

```go
package main

import "fmt"

type I interface {
    M()
}

func main() {
    var i I
    describe(i)
    i.M()
}

func describe(i I) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

## Empty interfaces

If you have an interface with no methods at all that is an **empty interface** `interface{}`.  An empty interface is a way to express that object could be anything.  Empty interfaces are used by code that handles values of unknown type.  For example `fmt.Print` takes any number of interfaces of type `interface{}`

```go
package main

import "fmt"

func main() {
    var i interface{}
    describe(i)

    i = 42
    describe(i)

    i = "hello"
    describe(i)
}

func describe(i interface{}) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

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
    // if the assertion is true, k will be assigned the value if it fails
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

A type assertion provides access to an interface value's underlying concrete type.  `t := i.(T)` This statement asserts that the interface value `i` holds the concrete type `T` and assigns the underlying `T` value to the variable `t`.  If `i` does not hold a `T`, the statement will trigger a panic.  To test whether an interface value holds a specific type, a type assertion can return two values: the underlying value and a boolean value that reports whether the assertion succeeded. `t, ok := i.(T)` If `i` holds a `T`, then `t` will be the underlying value and `ok` will be true.  If not, `ok` will be false and `t` will be the zero value of type `T`, and no panic occurs.

```go
package main

import "fmt"

func main() {
    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s)

    s, ok := i.(string)
    fmt.Println(s, ok)

    f, ok := i.(float64)
    fmt.Println(f, ok)

    f = i.(float64) // panic
    fmt.Println(f)
}
```

### Type switches with interfaces

A **type switch** is like a regular switch statement, but the cases in a type switch are specifically types (not values), and those value are compared against the type of the value held by the given interface value.  The declaration in a type switch has the same syntax as a type assertion `i.(T)`, but the specific type `T` is replaced with the keyword `type`.  The switch statement tests whether the interface value `i` holds a value of type `T` or `S`.  In the default case (where there is no match), the variable `v` is of the same interface type and value as `i`.

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

-------------------------
### The empty interface


### Type assertions

A type assertion provides access to an interface values's underlying concrete type.

`t := i.(T)`

This statement asserts that the interface value `i` holds the concrete type `T` and assigns the underlying `T` value to the variable `t`.  If `i` does not hold a `T`, the statement will trigger a panic.  To test whether an interface value holds a specific type, a type assertion can return two values: the underlying value and a boolean value that reports whether the assertion succeeded.

`t, ok := i.(T)`

If `i` holds a `T`, then `t` will be the underlying value and `ok` will be true.  If not, `ok` will be false and `t` will be the zero value of type `T`, and no panic occurs.  

```go
package main

import "fmt"

func main() {
    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s)

    s, ok := i.(string)
    fmt.Println(s, ok)

    f, ok := i.(float64)
    fmt.Println(f, ok)

    f = i.(float64) // panic
    fmt.Println(f)
}
```

## Stringer Interface

One of the most ubiquitous interfaces is `Stringer` defined int he `fmt` package.

```go
type Stringer interface {
    String() string
}
```

A `Stringer` is a ty pe that can describe itself as a string.  The `fmt` package and many others look for this interface to print values.

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
    a := Person{"Arthur Dent", 42}
    z := Person{"Zaphod Beeblebrox", 9001}
    fmt.Println(a, z)
}
```

## Generics

Generics is a feature added in Go 1.18 that lets you use a set of types as an interface.

```go
package main

import "fmt"

type Ordered interface {
    int | float64 | string
}

func min[T Ordered](values []T) (T, error) {
    if len(values) == 0 {
        var zero T
        return zero, fmt.Errorf("min of empty slice")
    }

    m := values[0]
    for _, v := range values[1:] {
        if v < m {
            m = v
        }
    }

    return m, nil
}

func main() {
    fmt.Println(min([]float64{2, 1, 3}))
    fmt.Println(min([]string{"B", "A", "C"}))
}
```
