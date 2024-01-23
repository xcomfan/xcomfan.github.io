---
layout: page
title: "Go Strings"
permalink: /go/types/strings
---

[comment]: <> (TODO: Need to add to this section printing strings and formatting of printing, and deatails of fmt package once I have those)

## Declaring Strings

You can declare strings in Go using **interpreted literals** or **raw string literals**.

### Interpreted Literals

An interpreted string must be a single line.  In an interpreted sting `\n` can be used for new line, `\t` for tab and `\` is the escape character.

```go
interpretedString := "Hello World!\n"
```

### Raw String Literals

Raw strings are surrounded with a back quote character.  They can span multiple lines, but there is no interpretation.  Each character in a raw sting literal is used literally.  You cannot use a back quote in a raw sting.

```go
rawString := `This is a raw string literal.  It can contain any character except backticks.`
```

## Zero value for stings

The zero value for stings is the empty string `""`.

## Concatenating Strings

You can concatenate strings with the `+` operator

```go
package main

import "fmt"

func main() {
    s := "Hello"
    s2 := "World"
    s3 := s + " " + s2 + "!"
    fmt.Println(s3)
}
```

## Getting length of a string

You can get the length of a string (in bytes) using the build in `len()` function.

## String Slicing

Go treats a string as *unmodifiable* sequence of bytes.  You can assess the bytes by slicing.  ***Note:** if your string is not in ASCII, accessing using bytes won't work well and you should look at the rune type.

[comment]: <> (TODO: Link to rune type above when you hae that written up)

You can slice characters and sections of a string.

```go
package main

import "fmt"

func main() {
    s := "Hello, world!"
    b := s[0]
    b2 := s[4]
    fmt.Println(s, b, string(b), b2, string(b2)) // prints Hello, world! 72 H 111 o

    s2 := s[0:5]
    fmt.Println(s, s2) // prints Hello world! Hello

    fmt.Println(len(s)) // prints 13 (number of bytes not chars)
}
```