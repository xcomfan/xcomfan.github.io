---
layout: page
title: "Go Maps"
permalink: /go/data_structures/maps
---

[comment]: <> (TODO: I should rework this section to be more common operation on Maps focused)

## Basic map operations

Maps in Go allow you to map a key to a value.  The value can be anything, but keys cannot be slices, maps or functions.

You cannot count on values in a map being in any sort of order; the order will be random.

### Insert and update

`m[key] = elem`

### Retrieve

`elem = m[key]`

### Delete

`delete(m, key)`

### Test that key is present

Test that a key is present with a two-value assignment

`elem, ok = m[key]`

If `key` is in `m` `ok` is `true` if not `ok` is `false`

If `key` is not in `m` then `elem` is the zero value for the map's element type.

Note: if `elem` or `ok` have not been declared you can use the short declaration `elem, ok := m[key]`

### Example of basic map operations

```go
package main

import "fmt"

func main() {
    m := make(map[string]int)

    m["Answer"] = 42
    fmt.Println("The value:", m["Answer"])

    m["Answer"] = 48
    fmt.Println("The value:", m["Answer"])

    delete(m, "Answer")
    fmt.Println("The value:", m["Answer"])

    v, ok := m["Answer"]
    fmt.Println("The value:", v, "Present?", ok)
}
```

#### Basic map operations with a struct

```go
package main

import "fmt"

type Vertex struct {
    Lat, Long float64
}

var m map[string]Vertex

func main() {
    m = make(map[string]Vertex)
    m["Bell Labs"] = Vertex{
        40.68433, -74.39967,
    }
    fmt.Println(m["Bell Labs"])
}
```

## Defining a map literal

```go
package main

import "fmt"

func main() {
    // define using map literal format.
    m2 := map[string]int{
    "a": 1,
    "b": 2,
    // each line needs to end in a comma even the last one
    "c": 50,
    }

    // you cannot count on values in a map being in any order its random each time
    for k, v := range m2 {
    fmt.Println(k, v)
    }

    fmt.Println("b in m2:", m2["b"])
    // delete value from map
    delete(m2, "b")
    fmt.Println("b in m2:", m2["b"])
}
```

### Defining map literal with struct values

```go
package main

import "fmt"

type Vertex struct {
    Lat, Long float64
}

var m = map[string]Vertex{
    "Bell Labs": Vertex{
        40.68433, -74.39967,
    },
    "Google": Vertex{
        37.42202, -122.08408,
    },
}

func main() {
    fmt.Println(m)
}
```

If the top-level type is just a type name, you can omit it from the elements of the literal.

```go
package main

import "fmt"

type Vertex struct {
    Lat, Long float64
}

var m = map[string]Vertex{
    "Bell Labs": {40.68433, -74.39967},
    "Google":    {37.42202, -122.08408},
}

func main() {
    fmt.Println(m)
}
```

## Nill maps and maps as reference types

Nill maps if passed to delete will do nothing.  Reading from a `nill` map will give you the zero value.  Trying to add a value to a `nill` map will cause a program to panic.

```go
package main

import "fmt"

func main() {
    m := map[string]int{
    "a": 1,
    "b": 2,
    }

    // initialize a map to its zero value
    var m3 map[string]int 

    fmt.Println("goodbye in m:", m["goodbye"])
    // maps are reference types so assigning one to another means they both pointing at same memory
    m3 = m 
    m3["goodbye"] = 300
    fmt.Println("goodbye in m3:", m3["goodbye"])
    fmt.Println("goodbye in m:", m["goodbye"])

    modMap(m)
    fmt.Println("cheese in m:", m["cheese"])
    }

    func modMap(m map[string]int) {
    m["cheese"] = 20
}
```
