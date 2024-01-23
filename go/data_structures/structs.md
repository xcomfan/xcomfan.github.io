---
layout: page
title: "Go Structs"
permalink: /go/data_structures/structs
---

## What are structs

[comment]: <> (TODO: Revisit this and write a better worded easier to follow definition.)

A struct is a collection of fields.  Structs are not objects.  There is no inheritance for structs or in Go in general.  There is however embedding and delegation.  

## Declaring structs

You declare a struct with the `struct` keyword.  Structs whose name starts with a capital letter are visible outside their packages.

```go
package main

import "fmt"

type Foo struct {
    // List the fields of the struct. They can be of any type. 
    A int   
    // variables starting with lower case are only visible within their packages. 
    b string
}

func main() {
    // initialize f to the zero value of the struct 
    // (the zero value of all members)
    f := Foo{}
    fmt.Println(f) // yields {0 } zero and empty string

    // define with values specified order 
    // must match the order they are declared in in the struct
    g := Foo{10, "Hello"} 
    // this type of struct declaration is not common because 
    // you need to refer back to struct definition to know the order
    // and must specify all the values all of the time.
    fmt.Println(g) // yields {10 Hello}

    // The more common way to initialize a struct.
    // Any unnamed values will stay at their zero value.
    h := Foo{
        b: "Goodbye",
    }

    fmt.Println(h) // yields {0 Goodbye}

    // Assign and read values from the struct
    h.A = 1000
    fmt.Println(h.A)
    fmt.Println(h.b)
}
```

## Reading values from a struct

Struct fields are accessed using the `.` operator.

Struct fields can be accessed through a struct pointer.  You can write `(*p).X` however that notation is cumbersome so Go lets you just write p.X without the explicit dereference.

```go
package main

import "fmt"

type Vertex struct {
    X int
    Y int
}

func main() {
    v := Vertex{1, 2}
    p := &v
    p.X = 1e9
    fmt.Println(v)
}
```

## Struct Literals

You can use a subset of fields using Name: syntax to define a struct literal.  Order of named fields is irrelevant.

The prefix `&` returns a pointer to the struct.

```go
package main

import "fmt"

type Vertex struct {
    X, Y int
}

var (
    v1 = Vertex{1, 2}  // has type Vertex
    v2 = Vertex{X: 1}  // Y:0 is implicit
    v3 = Vertex{}      // X:0 and Y:0
    p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
    fmt.Println(v1, p, v2, v3)
}
```

## Embedded structs

```go
package main

import "fmt"

type Foo struct {
    A int
    b string
}

type Bar struct {
    C string
    F Foo // field of type Foo
}

// Baz is not a subtype of Foo it just contains a Foo as a field.
type Baz struct {
    D   string
    // this is an embedded struct no name assigned to it.
    Foo 
}

func main() {
    f := Foo{A: 10, b: "Hello"}
    b1 := Bar{C: "Fred", F: f}
    fmt.Println(b1.F.A)
    b2 := Baz{D: "Nancy", Foo: f}
    // don't need to supply the intermediate struct.  Think of embedding an address for example.
    fmt.Println(b2.A) 

    var f2 Foo = b2.Foo
    fmt.Println(f2)
}
```

[comment]: <> (TODO: I think I need to move methods to their own section away from structs)


## Struct tags

[comment]: <> (TODO: Need to flesh out the struct tag thing)

Below is an example of converting a struct back and forth from JSON

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name string `json:"name"` // the string literal is the struct tag.
    Age  int    `json:age"`
}

func main() {
    bob := `{"name": "Bob", "age": 30}`
    var b Person
    // pass the json as a series of bytes along with a pointer to b
    json.Unmarshal([]byte(bob), &b)
    fmt.Println(b)
    // Marshall returns a slice of bytes and an error we are ignoring.
    bob2, _ := json.Marshal(b) 
    fmt.Println(string(bob2))
}
```




