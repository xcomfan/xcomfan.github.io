---
layout: page
title: "Go Numeric Types"
permalink: /go/types/numeric_types
---

## Special types

In Go `byte`, `int`, and `uint` are aliases for the specific types to provide a shorthand.  The 32 versus 64 is determined by your CPU architecture.

| Type Name | Same as |
| --------- | ------- |
| byte | uint8 |
| int | int32 or int64 |
| uint | uint32 or uint64 |

## Integer types

The zero value for integer types is zero `0`

| Type Name | Min Value | Max Value |
| --------- | --------- | --------- |
| int8 | -128 | 127 |
| int16 | -32,768 | 32,767 |
| int32 | -2,147,483,648 | 2,147,483,648 |
| int64 | -9,223,372,036,854,775,808 | -9,223,372,036,854,775,808 |
| uint8 | 0 | 255 |
| uint16 | 0 | 65,535 |
| uint32 | 0 | 4,294,967,293 |
| uint64 | 0 | 18,446,744,073,709,551,615 |


## Floating point types

The zero value for float types is zero `0`

| Type Name | Number of Bits | Number of Decimal Digits |
| float32 | 32 | ~7 |
| float64 | 64 | ~15 |

### Parsing Floats

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    f1, f2, f3 := 23.5, 65.1, 76.3
    floatSum := f1 + f2 + f3
    // prints x.yyyyyyyyy due to binary storage
    fmt.Println("Float sum:", floatSum)

    // *100 / 100 needed because by default math.Round will give integer
    //This is the safest and easiest way in Go to round numbers adn keep the decimal portion.
    // the *100 / 100 will give you 2 decimals of precisions (the number of 0s in 100) 
    // for 3 use 1000
    floatSum = math.Round(floatSum*100) / 100
    fmt.Println("The sum is now", floatSum)
}
```
