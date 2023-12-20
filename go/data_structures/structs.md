---
layout: page
title: "Go Structs"
permalink: /go/data_structures/structs
---

## Structs are not objects

There is no inheritance for structs or in Go in general.  There is however embedding and delegation.

## Declaring structs and assigning values

```go
package main

import "fmt"

// `type name struct` is how you declare a struct
// Structs whose name start with capital 
// letter are visible outside their packages
type Foo struct {
    // List fields of the struct. They can be of any type. 
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

## Methods on structs by reference and by value

Think of methods as functions that are part of the definition of a data type

```go
package main

import "fmt"

type Foo struct {
    A int
    B string
}

// this is a method declaration.
// It looks like a function declaration but has method
// receiver between func and name of method.
// the f here like self in Python
func (f Foo) String() string {
    return fmt.Sprintf("A: %d and B: %s", f.A, f.B)
}

// Method declaration but instead of being declared on Foo its being declared
// on pointer to Foo
// NOTE: if we declared it on Foo a copy would be passed into the double method
// and we would not be able to update the value.
// In general you should declare using value receiver if you are not modifying
// values in struct and reference receiver if you are modifying. 
func (f *Foo) Double() {
    f.A = f.A * 2
}

func main() {
    f := Foo{
        A: 10,
        B: "Hello",
    }
    fmt.Println(f.String())
    f.Double() // you can call a reference or value receiver from a reference that is not a pointer.
    fmt.Println(f.String())
}
```

The compiler won't allow you to call a reference receiver if the struct has not been assigned to a variable.  If you call value reciever on a pointer that is `nil`, the program will compile but panic at runtime.

### How embedding interacts with methods

***Note:*** in example below we are using embedding NOT inheritance

```go
package main

import "fmt"

type Foo struct {
    A int
    B string
}

func (f Foo) String() string {
    return fmt.Sprintf("%d fields: A: %d and B: %s", f.FieldCount(), f.A, f.B)
}

func (f *Foo) Double() {
    f.A = f.A * 2
}

func (f *Foo) FieldCount() int {
    fmt.Println("Foo version of FieldCount")
    return 2
}

type Bar struct {
    C   bool
    Foo // Bar has embedded struct of type Foo
}

func (b Bar) fieldCount() int {
    fmt.Println("Bar version of field count")
    return 3
}

func main() {
    f := Foo{
        A: 10,
        B: "Hello",
    }
    b := Bar{
        C:   true,
        Foo: f,
    }
    fmt.Println(b.String())
    b.Double()
    fmt.Println(b.String()) // Field count of Foo runs
}
```

### Methods on concrete types

Go allows you to create concrete types other than structs and define methods on these types like you would with structs.

```go
package main

import "fmt"

type myInt int

func (mi myInt) isEven() bool {
    return mi%2 == 0
}

func (mi *myInt) Double() {
    *mi = *mi * 2
}

func main() {
    m := myInt(10)
    fmt.Println(m)
    fmt.Println(m.isEven())
    m.Double()
    fmt.Println(m)
}
```

## Struct tags

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
