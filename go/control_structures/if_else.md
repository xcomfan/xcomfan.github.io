---
layout: page
title: "Go If Else Statement"
permalink: /go/control_structures/if_else
---

```go
package main

import "fmt"

func main(){
    a := 10
    if a > 5 { // Note: there are no parentheses around condition
        fmt.Println("a is bigger than 5")
    } else {
        fmt.Println("a is less than or equal to 5")
    }
}
```

You can declare a variable as part of the if statement and this variable will only be accessible in the scope of the if statement.  It does not have to be a strict variable declaration you can put a short expression there.  Variables declared inside an `if` short statement are also visible inside any of the else blocks.

```go
package main

import "fmt"

func main() {
    a := 10
    if b := a / 2; b > 5 {
        fmt.Println("b is bigger than 5:", a, b)
    } else {
        fmt.Println("b is less than or equal to 5:", a, b)
    }
    fmt.Println(b) // This will give you undefined error, but b ok to use above.
}
```

```go
package main

import (
    "fmt"
    "math"
)

func pow(x, n, lim float64) float64 {
    if v := math.Pow(x, n); v < lim {
        return v
    } else {
        fmt.Printf("%g >= %g\n", v, lim)
    }
    // can't use v here, though
    return lim
}

func main() {
    fmt.Println(
        pow(3, 2, 10),
        pow(3, 3, 20),
    )
}
```
