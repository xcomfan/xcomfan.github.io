---
layout: page
title: "Go Slices"
permalink: /go/data_structures/slices
---

## Basic slice example

```go
package main

import "fmt"

func main() {
    // []string is a slice declaration make is used to create certain built in 
    // types in Go
    s := make([]string, 0)
    fmt.Println("length of s:", len(s))
    // append adds new element to the end of a slice.  
    // NOTE: you must assign the append output back to your slice or slice
    // does not get updated.
    s = append(s, "hello") 
    fmt.Println("length of s:", len(s))
    fmt.Println("content of s[0]", s[0])
    s[0] = "goodbye"
    fmt.Println("contents of s[0]:", s[0])

    s2 := make([]string, 2)
    fmt.Println("contents of s2[0]:", s2[0])
    // adding third element here.
    s2 = append(s2, "hello")
    fmt.Println("contents of s2[0]:", s2[0]) 
    fmt.Println("contents of s2[2]:", s2[2])
    // length is now 3 because append left the first two elements as is.
    fmt.Println("length of s2:", len(s2))
}
```

## Slice literal assignment, iteration and assignment

```go
package main

import "fmt"

func main() {
    // initializing using slice literal format
    s3 := []string{"a", "b", "c"}

    // // k gives you the index and v is the value
    for k, v := range s3 {
    fmt.Println(k, v)
    }

    s4 := s3[0:2]
    fmt.Println("s4:", s4)
    // if you take a slice from a slice they both refer to same area of 
    // memory so changing one impacts the other.
    // Slices are reference types. They behave like pointers do.
    s3[0] = "d" 
    fmt.Println("s4:", s4)

    var s5 []string
    s5 = s3
    s5[1] = "camel"
    fmt.Println("s3:", s3)

    modSlice(s3)
    fmt.Println("s3[0]:", s3[0])
    fmt.Println("s3:", s3)
    }

    func modSlice(s []string) {
    s[0] = "hello"
}
```