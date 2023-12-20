---
layout: page
title: "Go Functions"
permalink: /go/functions
---

## Defining a Function

Go does not have optional or named parameters.  Go also does ot do function overloading.  

***Note:** All arguments in Go are passed by value.  A copy of the object being passed is made.

```go
func addNumbers(a int, b int) int {
    return a + b
}
```

## Functions can return multiple values

```go
func divAndRemainder(a int, b int) (int, int){
    return a / b, a % b
}

div, remainder = divAndRemainder(2, 3)
fmt.Println(div, remainder)

_, rem = divAndRemainder(2, 3) // convention to use _ to use variable you don't need as go makes you read variables that are declared.
```

## Functions Can Be Passed As Arguments

This is useful when you want to register a function to execute when a certain event occurs.  The example of the Go http server where you define a function that executes when a certain route is requested is an example of this.

```go
package main

import "fmt"

func addOne(a int) int {
    return a + 1
}

func addTwo(a int) int {
    return a + 2
}

func printOperation(a int, f func(int) int){ // Any function that takes int arg and returns int can be passed in here.
    fmt.Println(f(a)) // here we are calling the function that was passed ot us.
}

func main(){
    //passing in the two functions
    printOperation(1, addOne)
    printOperation(1, addTwo)
}
```

You can also return a function from a function.

```go
package main

import "fmt"

func makeAdder(b int) func(int) int {
    return func(a int) int {
        return a + b  // b is stored in the closure and a is arg passed when function is called.
    }
}

func main(){
    addOne := makeAdder(1)
    addTwo := makeAdder(2)

    fmt.Println(addOne(1))
    fmt.Println(addTwo(1))
}
```

Putting it all together; you can have a function that takes a function as an argument and returns a function.  This is useful if you need to run code around a function.  For example `http.HandleFunc("/hello", log(printHello))` `http.HandleFunc("/bye", log(printBye))` where `printHello` and `printBye` are their own respective functions.

```go
package main

import "fmt"

func makeAdder(b int) func(int) int {
    return func(a int) int {
        return a + b
    }
}

func makeDoubler(f func(int) int) func(int) int {
    return func(a int){
        b := f(a)
        return b * 2
    }
}

func main(){
    addOne := makeAdder(1)
    doubleAddOne := makeDoubler(addOne)

    fmt.Println(addOne(1))
    fmt.Println(doubleAddOne(1))
}
```

## Anonymous Functions

You can declare a function inside of another function using anonymous functions.  A function declared this way is not accessible outside the function it is declared in and the variables inside the anonymous functions are not accessible in the enclosing function.  Variables of the "outer" function are available in the "inner" function and thus you have closures.  You CAN assign values to the variables of the outer function from the inner function.

```go
package main

import "fmt"

func main(){
    myAddOne := func(a int) int {
        return a + 1
    }
    fmt.Println(myAddOne(1))
}
```