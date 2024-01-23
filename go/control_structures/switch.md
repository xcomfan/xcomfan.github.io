---
layout: page
title: "Go Switch Statement"
permalink: /go/control_structures/switch
---

## Basic switch usage

The `switch` statement is a shorthand way to write a sequence of `if else` statements.  In Go unlike other languages you do not need a `break` in each case.  Go will run just the one case unless you use the `fallthrough` option.  Go will run the first case whose value is equal to the condition expression.

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    word := os.Args[1] // get argument from command line
    switch word {
    case "hello":
        fmt.Println("Hi yourself")
    case "goodbye":
        fmt.Println("So long!")
    case "greetings":
        fmt.Println("Salutations!")
    default:
        fmt.Println("I don't know what you said")
    }
}
```

## Applying the same code for multiple cases

If you wish to run the same code for two or more cases you can separate the checks with a comma

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    word := os.Args[1] // get argument from command line
    switch word {
    case "hi":
        fmt.Println("Very informal!")
        fallthrough
    case "hello":
        fmt.Println("Hi yourself")
    case "goodbye", "bye": // same code runs for both values.
        fmt.Println("So long!")
    case "greetings":
        fmt.Println("Salutations!")
    default:
        fmt.Println("I don't know what you said")
    }
}
```

## Variable with scope of switch statement

You can also define a variable that will only exist in the scope of the switch statement as part of the declaration.

```go
...
word := os.Args[1]
switch l := len(word); word {
    case "hi":
        ...
}
...
```

## Using fallthrough to run next case

By default Go will only run the case that matches.  If you want Go to run the next case (it will just run the single next case) use the `fallthrough` keyword.

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    word := os.Args[1] // get argument from command line
    switch word {
    case "hi":
        fmt.Println("Very informal!")
        fallthrough
    case "hello":
        fmt.Println("Hi yourself")
    case "goodbye":
        fmt.Println("So long!")
    case "greetings":
        fmt.Println("Salutations!")
    default:
        fmt.Println("I don't know what you said")
    }
}
```

The above code execution show below...

```text
go run string.go hi
Very informal!
Hi yourself
```

## Switch with no condition

Switch without a condition is the same as `switch true`.  This construct can be a clean way to write long if-then-else chains

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    t := time.Now()
    switch {
    case t.Hour() < 12:
        fmt.Println("Good morning!")
    case t.Hour() < 17:
        fmt.Println("Good afternoon.")
    default:
        fmt.Println("Good evening.")
    }
}
```

## Case with no code

It is allowed in Go to have a case with no code.  It is treated as a no op.

```go
 package main

import (
    "fmt"
    "os"
)

func main() {
    word := os.Args[1] // get argument from command line
    switch word {
    case "hi":
        fmt.Println("Very informal!")
        fallthrough
    case "hello":
    case "farewell": // can have a no op case if you need.  Nothing will be printed if this clause matches.
        fmt.Println("Hi yourself")
    case "goodbye", "bye":
        fmt.Println("So long!")
    case "greetings":
        fmt.Println("Salutations!")
    default:
        fmt.Println("I don't know what you said")
    }
}
```
