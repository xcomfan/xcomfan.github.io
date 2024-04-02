---
layout: page
title: "Go Functions"
permalink: /go/functions
---

[comment]: <> (TODO: This is a good candiate for breaking up with maybe function arguments getting their own section as that is pretty involved)

## Defining a Function

Go does not have optional or named parameters.  Go also does not do function overloading.  

***Note:** All arguments in Go are passed by value.  A copy of the object being passed is made.  You can use pointers to pass by reference.

```go
func addNumbers(a int, b int) int {
    return a + b
}
```

When two or more consecutive named function parameters share a type, you can omit the type from all but the last.  Thus the above example can be written as.

```go
func addNumbers(a, b int) int {
    return a + b
}
```

## Functions can return any number values

```go
func divAndRemainder(a int, b int) (int, int){
    return a / b, a % b
}

div, remainder = divAndRemainder(2, 3)
fmt.Println(div, remainder)

_, rem = divAndRemainder(2, 3) // convention to use _ for variable you don't need as go makes you read variables that are declared.
```

## Naked return

Go's return values may be named.  You do this by defining them at the top of the function.  These names should be used to document the meaning of the return values.  A `return` statement without arguments returns the named return values.  This is known as the **naked return**.

```go
package main

import "fmt"

func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}

func main() {
    fmt.Println(split(17))
}
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

## Function closures

Go functions may be closures.  A closure is a function value that references variables from outside its body.  The function may access and assign to the referenced variables; in this sense the function is bound to the variables.

In example below, the `adder` function returns a closure.  Each closure is bound to its own `sum` variable.

```go
package main

import "fmt"

func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}

func main() {
    pos, neg := adder(), adder()
    for i := 0; i < 10; i++ {
        fmt.Println(
            pos(i),
            neg(-2*i),
        )
    }
}
```

## Type parameters

Go functions can be written to work on multiple types using type parameters.  The type parameters of a function appear between brackets before the function's arguments.

```go
func Index[T comparable](s []T, x T) int
```

This declaration means that `s` is a slice of any type `T` that fulfills the built-in constraint `comparable`.  `x` is also a value of the same type.

`comparable` is a useful constraint that makes it possible to use the `==` and `!=` operators on values of the type.  

In the example below we use it to compare a value to all slice elements until a match is found.  This index function works for any type that supports comparison.

```go
package main

import "fmt"

// Index returns the index of x in s, or -1 if not found.
func Index[T comparable](s []T, x T) int {
    for i, v := range s {
        // v and x are type T, which has the comparable
        // constraint, so we can use == here.
        if v == x {
            return i
        }
    }
    return -1
}

func main() {
    // Index works on a slice of ints
    si := []int{10, 20, 15, -10}
    fmt.Println(Index(si, 15))

    // Index also works on a slice of strings
    ss := []string{"foo", "bar", "baz"}
    fmt.Println(Index(ss, "hello"))
}
```


```
