---
layout: page
title: "Go Slices"
permalink: /go/data_structures/slices
---

[comment]: <> (TODO: This page needs a reorganization)

## What are slices

An array in Go is fixed in size, but a slice is a dynamically-sized flexible view into the elements of an array.

A slice does not store any data, it just describes a section of an underlying array.  Changing the elements of a slice modifies the corresponding elements of its underlying array.  Other slices that share the same underlying array will see those changes.

`[]T` is a slice with elements of type `T`

A slice is formed by specifying two indices a low and a high.  This selects a half-open range which includes the first element, but excludes the last one.  The following expression creates a slice which includes elements 1 through 3 of `a`.  You can omit the high and low bounds to get their defaults of 0 lower bound and length of slice as upper bound.

`a[1:4]`

A slice has both a length and a capacity.

The **length** of a slice is the number of elements it contains.

The **capacity** of a slice is the number of elements in the underlying array, counting from the first element in the slice.

The length and capacity of a slice can be obtained using the expressions len(s) and cap(s).

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

## Appending to a slice

The build in `append` function is used to append elements to a slice.

`func append(s, []T, vs ...T)[]T`

The first parameter `s` is a slice of type `T`, and the rest are `T` values to append to the slice.  The resulting value of `append` is a slice containing all the elements of the original slice plus the provided values.  If the backing array of `s` is too small to fit all the given values a bigger array will be allocated.  The returned slice will point to the newly allocated array.

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

## Iterating over a slice

To iterate over a slice use the `for range` loop to iterate over a slice.

```go
package main

import "fmt"

func main() {
    // initializing using slice literal format
    s := []string{"a", "b", "c"}

    // k gives you the index and v is the value
    for k, v := range s {
        fmt.Println(k, v)
    }
}
```

## Packaging

You should have a comment before all exported functions (ones that start with a capital)

## Working with files and the web

```go
package main

import (
    "fmt"
    "io"
    "io/ioutil"
    "os"
)

func main() {
    content := "Hello from Go!"
    file, err := os.Create("./fromString.txt")
    checkError(err)
    length, err := io.WriteString(file, content)
    checkError(err)
    fmt.Printf("Wrote a file with %v chars\n", length)
    defer file.Close()
    defer readFile("./fromString.txt")
}

func readFile(fileName string) {
    data, err := ioutil.ReadFile(fileName)
    checkError(err)
    fmt.Println("Text read from file:", string(data))
}

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}
```

## Parsing a JSON without declaring the struct ahead of time.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strings"
)

const url = "http://services.explorecalifornia.org/json/tours.php"

func main() {

    resp, err := http.Get(url)
    if err != nil {
        panic(err)
    }

    fmt.Printf("Response type: %T\n", resp)

    defer resp.Body.Close()

    bytes, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        panic(err)
    }

    content := string(bytes)
    // fmt.Print(content)

    tours := toursFromJson(content)
    for _, tour := range tours {
        fmt.Println(tour.Name)
    }
}

func toursFromJson(content string) []Tour {
    tours := make([]Tour, 0, 20)

    decoder := json.NewDecoder(strings.NewReader(content))
    _, err := decoder.Token()
    if err != nil {
        panic(err)
    }

    var tour Tour
    for decoder.More() {
        err := decoder.Decode(&tour)
        if err != nil {
        panic(err)
        }
        tours = append(tours, tour)
    }
    return tours
}

type Tour struct {
    Name, Price string
}
```
