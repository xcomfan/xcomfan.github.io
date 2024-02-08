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

## Returning formatted error with fmt package

The `fmt` package has a function that lets you return a formatted string as an error.

```go
package main

import (
    "fmt"
    "math"
)

func sqrt(n float64) (float64, error) {
    if n < 0 {
        return 0.0, fmt.Errorf("sqrt of negative value (%f)", n)
    }
    return math.Sqrt(n), nil
}

func main() {
    _, err := sqrt(-1)
    fmt.Printf("Type of err = %\n", err) // same as you would get with errors.New()
}
```

## Error are an interface

Because error is an interface, we can define our own instead of relying on ones given to us by `errors.New()`

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


## Errors package

[comment]: <> (TODO: The package being discussed here did not work for me and the code is not validated.  I will revisit once I get more clarity on how to deal with third party packages)

There is a commonly used package called pkg errors.  This is a drop in replacement to the built in pkg/errors package.

Below is a good example of how to open up a config file and handle the possibility of it being missing as well as some basic logging.

```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/pkg/errors"
)

// Config holds configuration
type Config struct {
    // configuration fields go here (redacted)
}

func readConfig(path string) (*Config, error) {
    file, err := os.Open(path)
    if err != nil {
        return nil, errors.Wrap(err, "can't open configuration file")
    }

    defer file.Close()

    cfg := &Config{}
    // Parse file here (redacted)
    return cfg, nil
}

func setupLogging() {
    out, err := os.OpenFile("app.log", os.O_APPEND|os.O_CREATE|os.O_RDONLY, 0644)
    if err != nil {
        return
    }
    log.SetOutput(out)
}

func main() {
    setupLogging()
    // we expect this to fail since we don't have config file on system
    cfg, err := readConfig("/path/to/config.toml")
    if err != nil {
        fmt.Fprintf(os.Stderr, "error: %s\n", err)
        // %+v will print stack trace as part of the error package
        log.Printf("error: %+v", err)
        os.Exit(1)
    }

    // Normal operation redacted
    fmt.Println(cfg)
}
```

## Use defer and recover to guard against panics

To guard against a panic use `defer`. Inside `defer` you call the built in `recover()` which returns something.  If the something is `nil` there is no panic if its not `nil` you get an error.

Below is an example of using `defer` and `recover`

```go
func safeValue(vals []int, index int) (n int, err error){
    defer func() {
        if e := recover(); e != nil {
            err = fmt.Errorf("%v", e)
        }
    }()

    return vals[index], nil
}
```
