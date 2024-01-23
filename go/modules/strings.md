---
layout: page
title: "strings module"
permalink: /go/modules/fmt
---

## Overview

The `strings` package implements functions for manipulating UTF-8 encoded strings.

## Splitting a string

The function `func Fields(s string) []string` splits the string `s` around each instance of one or more consecutive white space characters returning a slice of substrings of `s` or an empty slice if `s` contains only white space.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    myString := "Von Wegen Lisbeth - Grande 2016"
    words := strings.Fields(myString)
    for _, word := range words {
        fmt.Println(word)
    }
}
```
