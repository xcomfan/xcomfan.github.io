---
layout: page
title: "Go Type Conversion"
permalink: /go/types_conversion
---

## Go Does Not Convert Types Automatically

The code below will **NOT** compile and give you the error `invalid operation: i + f (mismatched types int8 and float32)`

```go
package main
import "fmt"

func main(){
    var i int8 = 20
    var f float32 = 5.6
    fmt.Println(i + f) // NOT VALID
}
```

You need to use the conversion syntax such as `float32(i)` to convert to the desired types.  ***NOTE:*** if you are converting a float to an int the value will be truncated not rounded

You don't just need to convert between float and int but different int sizes (`int16` and `int32` for example) must also be converted.

```go
package main

import "fmt"

func main() {
    var i int8 = 20
    var f float32 = 5.6
    fmt.Println(float32(i) + f) // Convert i to a float.  Prints 25.6
    fmt.Println(i + int8(f))    // Prints 25  NOTE: value is truncated not rounded
}
```
