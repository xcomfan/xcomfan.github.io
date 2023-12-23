---
layout: page
title: "Pointers in Go"
permalink: /go/pointers
---

## Zero value

The zero value of a pointer is `nil`

## Basic pointer example

```go
package main

import "fmt"

func main() {
    a := 10
    b := &a
    c := a

    fmt.Println(a, b, *b, c)

    a = 20
    fmt.Println(a, b, *b, c)

    *b = 30
    fmt.Println(a, b, *b, c)

    c = 40
    fmt.Println(a, b, *b, c)
}
```

above code prints

```text
10 0x400000e090 10 10
20 0x400000e090 20 10
30 0x400000e090 30 10
30 0x400000e090 30 40
```

## Pointers can be used to pass by reference to functions

```go
package main

import "fmt"

func setTo10(a *int) {
    *a = 10
}

func main() {
    a := 20
    fmt.Println(a)
    setTo10(&a)
    fmt.Println(a)
}
```

## Pointes to types

You can declare a variable as a pointer to another type.

```go
package main

import "fmt"

func main(){
    var b *int // does not point to memory location but absence of one (nill)

    fmt.Println(b) // will print nill to console
    fmt.Println(*b) // will cause a panic when this executes
}
```

You can also use the `new()` function to crate a pointer

```go
package main

import "fmt"

func main(){
    b := new(int) // new makes a pointer for the type and allocates memory

    fmt.Println(b) // prints memory location
    fmt.Println(*b) // prints the zero value of int of 0.
}
```

## Go versus C pointers

Pointers in Go are less scary than in C because...

* Go has built in array and string types.

* Go has garbage collection.

* Go is strongly types (can't cast one pointer type to another)

* Unlike C, go has no pointer arithmetic
