---
layout: page
title: "Go Slices"
permalink: /go/data_structures/slices
---

[comment]: <> (TODO: This page needs a reorganization)

## What are slices

An array in Go if fixed in size, but a slice is a dynamically-sized flexible view into the elements of an array.

A slice does not store any data, it just describes a section of an underlying array.  Changing the elements of a slice modifies the corresponding elements of its underlying array.  Other slices that share the same underlying array will see those changes.

`[]T` is a slice with elements of type `T`

A slice is formed by specifying two indices a low and a high.  This selects a half-open range which includes the first elment, but excludes the last one.  The following expression creates a slice which includes elements 1 through 3 of a:  You can omit the high and low bounds to get their defaults of 0 lower bound and length of slice as upper bound.

`a[1:4]`

A slice has both a length and a capacity.

The length of a slice is the number of elements it contains.

The capacity of a slice is the number of elements in the underlying array, counting from the first element in the slice.

The length and capacity of a slice s can be obtained using the expressions len(s) and cap(s).

You can extend a slice's length by re-slicing it, **provided it has sufficient capacity**.

```go
package main

import "fmt"

func main() {
    s := []int{2, 3, 5, 7, 11, 13}
    printSlice(s)

    // Slice the slice to give it zero length.
    s = s[:0]
    printSlice(s)

    // Extend its length.
    s = s[:4]
    printSlice(s)

    // Drop its first two values.
    s = s[2:]
    printSlice(s)
}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```

```go
package main

import "fmt"

func main() {
    names := [4]string{
        "John",
        "Paul",
        "George",
        "Ringo",
    }
    fmt.Println(names)

    a := names[0:2]
    b := names[1:3]
    fmt.Println(a, b)

    b[0] = "XXX"
    fmt.Println(a, b)
    fmt.Println(names)
}
```

## Nil slices

The zero value of a slice is `nil`.  A `nil` slice has a length and capacity of 0 and has no underlying array.

```go
package main

import "fmt"

func main() {
    var s []int
    fmt.Println(s, len(s), cap(s))
    if s == nil {
        fmt.Println("nil!")
    }
}
```

## Creating a slice with make

Slices can be created with the built-in make function; this is how you create dynamically-sized arrays.

The make function allocates a zeroed array and returns a slice that refers to that array:

```go
a := make([]int, 5)  // len(a)=5
```

To specify a capacity, pass a third argument to make:

```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

```go
package main

import "fmt"

func main() {
    a := make([]int, 5)
    printSlice("a", a)

    b := make([]int, 0, 5)
    printSlice("b", b)

    c := b[:2]
    printSlice("c", c)

    d := c[2:5]
    printSlice("d", d)
}

func printSlice(s string, x []int) {
    fmt.Printf("%s len=%d cap=%d %v\n",
    s, len(x), cap(x), x)
}
```

## slices of slices

Slices can contain any type, including other slices.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // Create a tic-tac-toe board.
    board := [][]string{
        []string{"_", "_", "_"},
        []string{"_", "_", "_"},
        []string{"_", "_", "_"},
    }

    // The players take turns.
    board[0][0] = "X"
    board[2][2] = "O"
    board[1][2] = "X"
    board[1][0] = "O"
    board[0][2] = "X"

    for i := 0; i < len(board); i++ {
        fmt.Printf("%s\n", strings.Join(board[i], " "))
    }
}
```

 Below is an example from the go tour of how to do a basic matrix with slices.

 ```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
    pic := make([][]uint8, dy)
    for y := range pic {
        pic[y] = make([]uint8, dx)
        for x := range pic[y] {
            pic[y][x] = uint8(x*y)
        }
    }
    return pic
}

func main() {
    pic.Show(Pic)
}
 ```

## Appending to a slice

The build in `append` function used to append elements to a slice has the format `func append(s, []T, vs ...T)[]T`

The first parameter `s` is a slice of type `T`, and the rest are `T` values to append to the slice.  The resulting value of `append` is a slice containing all the elements of the original slice plus the provided values.  If the backing array of s is too small to fit all the given values a bigger array will be allocated.  The returned slice will point to the newly allocated array.

```go
package main

import "fmt"

func main() {
    var s []int
    printSlice(s)

    // append works on nil slices.
    s = append(s, 0)
    printSlice(s)

    // The slice grows as needed.
    s = append(s, 1)
    printSlice(s)

    // We can add more than one element at a time.
    s = append(s, 2, 3, 4)
    printSlice(s)
}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

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