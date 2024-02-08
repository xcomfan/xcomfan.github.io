---
layout: page
title: "strings module"
permalink: /go/modules/fmt
---

## Overview

The `strings` package implements functions for manipulating UTF-8 encoded strings.  Full documentation can be found [here](https://pkg.go.dev/strings)

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

## Check if a substring is in a string

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println(strings.Contains("seafood", "foo")) // yields true
    fmt.Println(strings.Contains("seafood", "bar")) // yield false
    fmt.Println(strings.Contains("seafood", "")) // yields true
    fmt.Println(strings.Contains("", "")) // yields true
}
```

## Lower or uppercase a sting

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println(strings.ToLower("Gopher"))
    fmt.Println(strings.ToUpper("Gopher"))
}
```

## Convert strings to other types

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strconv"
    "strings"
)

func main() {

    reader := bufio.NewReader(os.Stdin)
    fmt.Print("Enter text: ")
    input, _ := reader.ReadString('\n')
    fmt.Println("You entered:", input)
    fmt.Print("Enter a number: ")
    numInput, _ := reader.ReadString('\n')
    aFloat, err := strconv.ParseFloat(strings.TrimSpace(numInput), 64)
    if err != nil {
        fmt.Println(err)
    } else {
    fmt.Println("Value of number: ", aFloat)
    }
}
```
