---
layout: page
title: "Go Arrays"
permalink: /go/data_structures/arrays
---

Arrays in Go are fixed in size, and the size of the array is part of the declaration.  For this reason you cannot assign arrays of different sizes to each other.

```go
package main

import "fmt"

func main(){
    var vals [3]int
    vals[0] = 2
    vals[1] = 4
    vals[2] = 6
    var vals2 [4]int = vals // NOT VALID GO CODE

    fmt.Println(vals)
}
```