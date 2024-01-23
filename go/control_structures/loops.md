---
layout: page
title: "Go Loops"
permalink: /go/control_structures/loops
---

## Go for loop has 3 forms

The `for` loop has three components separated by semicolons:

* the init statement.  Variables declared here are only visible within the scope of the for loop

* the condition expression: evaluated before every iteration.  The for loops stops executing once this condition is false.

* the post statement: executed at the end of every iteration

These components may be included or omitted to modify the behavior of the `for` loop.

### Normal for Loop

If all 3 elements are included you will get for loop behavior similar to other languages.

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

### Simulating While Loop Behavior

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

### Looping and controlling flow with break And continue

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

[comment]: <> (TODO: It may make sense to have iteration in its own section as its not exactly for loop specific.)

## Using for range To Iterate Over Data Structures

There is a `for range` loop which can iterate over some of Go's built in types such as arrays, slices, maps, strings and channels.  When ranging over a slice, two values are returned for each iteration.  The first is the index, and the second is a copy of the element at that index.

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

You can use the `_` variable name to skip the index or value if you do not need it.

```go
package main

import "fmt"

func main() {
    pow := make([]int, 10)
    for i := range pow {
        pow[i] = 1 << uint(i) // == 2**i
    }
    for _, value := range pow {
        fmt.Printf("%d\n", value)
    }
}
```

If you just want the index the format `for i := range pow` is valid as well.
