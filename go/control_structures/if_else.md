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

You can declare a variable as part of the if statement and this variable will only be accessible in the scope of the if statement.

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
