---
layout: page
title: "fmt module"
permalink: /go/modules/fmt
---

## Overview

The `fmt` package implements formatted I/O.  

[Full reference to `fmt` package](https://pkg.go.dev/fmt)

## Formatting Verbs

| Verb | Purpose/Type | Sample Use and Output |
| ---- | ------------ | --------------------- |
| `%v`| The value in a default format | `fmt.Printf("%v is %v years old\n", p.name, p.age)` -> Boris is 40 years old |
| `%#v` | A Go-syntax representation of the value | `fmt.Printf("%#v is %#v years old\n", p.name, p.age)` | "Boris" is 40 years old |
| `%T` | A Go-syntax representation of the type of the value | `fmt.Printf("%T is %T years old\n", p.name, p.age)` | string is int years old |
| `%t` | The word true or false | `fmt.Printf("status = %t\n", p.active)` | status = true |
| `%s` | The uninterpreted bytes of a string or slice | `fmt.Printf("String is %s\n", myString)` | String is ya ya ding dong |
| `%q` | a double quoted string safely escaped with Go syntax | `fmt.Printf("String is %q\n", myString)` | String is "ya ya ding dong" |

***Note:*** For compound operands such as slices and structs, the format applies to the elements of each operand recursively, not to the operand as a whole.  Thus `%q` for example will quote each element of a slice of strings and `%6.2f` will control formatting for each element of a floating-point array.

```go

package main

import (
    "fmt"
    "strings"
)

func main() {
// Outputs: Fields are: ["foo" "bar" "baz"]
    fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
}
```

## Printing keys and values of a struct

You can use the verb `%#v` to print a struct with its keys and values.

package main

import (
    "fmt"
)

type Person struct {
    Name string
    Age  int
}

func main() {
    boris := Person{
        Name: "Boris",
        Age:  40,
    }
    // will print boris = main.Person{Name:"Boris", Age:40}
    fmt.Printf("boris = %#v\n", boris)
}
