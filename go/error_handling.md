---
layout: page
title: "Error Handling in Go"
permalink: /go/error_handling
---

[comment]: <> (TODO: I would like to rewrite this section.  I am not happy with these examples and built up of concepts)

## Go does not have exceptions

Instead of exceptions Go returns values to determine if an error occurred.  This is more functional in Go than the way it was in C because you can return multiple values from a functions one of them being the error code.

***Note:*** the repetitiveness  in the below code is normal for Go.

```go
package main

import (
    "errors"
    "fmt"
    "os"
    "strconv"
)

func divAndMod(a int, b int) (int, int, error) {
    if b == 0 {
        return 0, 0, errors.New("Cannot divide by zero")
    }

    return a / b, a % b, nil
}

func main() {
    if len(os.Args) < 3 {
        fmt.Println("Expected two input parameters")
        os.Exit(1)
    }
    a, err := strconv.ParseInt(os.Args[1], 10, 64)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    b, err := strconv.ParseInt(os.Args[2], 10, 64)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    // parse int returns a 64 bit so we convert to the type divAndMod expects
    div, mod, err := divAndMod(int(a), int(b))
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Printf("%d / %d == %d and %d %% %d == %d\n", a, b, div, a, b, mod)
}
```

## Error are an interface

Because error is an interface, we can define ours instead of relying on ones given to us by `errors.New()`

```go
package main

import (
    "fmt"
    "os"
    "strconv"
)

type MyError struct {
    A       int
    B       int
    Message string
}

// Declare method error on a pointer to my error which matches signature for Error
func (me *MyError) Error() string {
    return fmt.Sprintf("values %d and %d produced error %s", me.A, me.B, me.Message)
}

func divAndMod(a int, b int) (int, int, error) {
    if b == 0 {
        //Instantiate a pointer to MyError and populate it.
        return 0, 0, &MyError{A: a, B: b, Message: "Cannot divide by zero"}
    }
    return a / b, a % b, nil
}

func main() {
    if len(os.Args) < 3 {
        fmt.Println("Expected two input parameters")
        os.Exit(1)
    }
    a, err := strconv.ParseInt(os.Args[1], 10, 64)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    b, err := strconv.ParseInt(os.Args[2], 10, 64)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    div, mod, err := divAndMod(int(a), int(b))
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Printf("%d / %d == %d and %d %% %d == %d\n", a, b, div, a, b, mod)

}
```

[comment]: <> (TODO: I should move this to interface section and link back to it here for explanation)

An interface is only `nil` if there is no underlying type assigned to it.  This is not specific to error interface, but all interfaces in Go.  Don't define a variable to be your own error type because if you return that variable it will not appear `nil`.

```go
package main

import "fmt"

func reallyNil() error {
    var e error
    fmt.Println("e is nil:", e == nil)
    return e
}

type MyError struct {
    A       int
    B       int
    Message string
}

func (me *MyError) Error() string {
    return fmt.Sprintf("values %d and %d produced error %s", me.A, me.B, me.Message)
}

func notReallyNil() error {
    var me *MyError
    fmt.Println("me is nil:", me == nil)
    return me
}

func main() {
    e := reallyNil()
    me := notReallyNil()
    fmt.Println("in main, e is nil:", e == nil)
    fmt.Println("in main, me is nil:", me == nil)
}
```