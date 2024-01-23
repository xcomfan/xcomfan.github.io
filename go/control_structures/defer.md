---
layout: page
title: "Go defer statement"
permalink: /go/control_structures/defer
---

## Basic usage

The defer statement defers the execution of a functions until the surrounding functions returns.  The deferred calls arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

In the below example "world" will be printed after "hello"

```go
package main

import "fmt"

func main() {
    defer fmt.Println("world")

    fmt.Println("hello")
}
```

## Stacking defers

Deferred function calls are pushed onto a stack.  When a function returns, its deferred calls are executed in last-in-first-out order.

The example below will print in backwards order due to the last-in-first-out nature.

```go
package main

import "fmt"

func main() {
    fmt.Println("counting")

    for i := 0; i < 10; i++ {
        defer fmt.Println(i)
    }

    fmt.Println("done")
}
```
