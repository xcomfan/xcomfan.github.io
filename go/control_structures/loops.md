---
layout: page
title: "Go Loops"
permalink: /go/control_structures/loops
---

## Basic for Loop

```go
package main

import "fmt"

func main(){
    a :=3
    for i := 0; i < 10; i++ {
        if i == a {
            continue
        }
        fmt.Println(i)
    }
}
```

## Using for range To Iterate Over Data Structures

There is a `for range` loop which can iterate over some of Go's built in types such as arrays, slices, maps, strings and channels.

```go
package main

import "fmt"

func main() {
    s := "Hello, world!"
    for k, v := range s { // k is the offset and v is the rune
        fmt.Println(k, v, string(v))
    }
}
```

## Simulating While Loop Behavior

In Go, instead of having a while statement the initializations and increment portions of the for loop are optional so you can use a for loop without those to have the behavior of a while loop.

```go
package main

import "fmt"

func main(){
    i := 0
    for i < 10 {
        fmt.Println(i)
        i = i + 1
    }
    fmt.Println(i)
}
```

## Looping And Controlling Flow with break And continue

You can also leave the condition off altogether and use `break` and `continue` for control flow.

```go
package main

import "fmt"

func main(){
    i := 0
    for {
        fmt.Println(i)
        i = i + 1
        if i > 10{
            break
        }
    }
    fmt.Println(i)
}
```
