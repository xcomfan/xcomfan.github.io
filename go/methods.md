---
layout: page
title: "Go Methods"
permalink: /go/methods
---

## Methods on structs

Go does not have classes, but you can define Methods on types.  A method is a function with a special receiver argument.  The receiver appears in its own argument list between the `func` keyword and the method name.  In the example below the `Abs` method has a receiver of type Vertex named v.

The compiler won't allow you to call a reference receiver if the struct has not been assigned to a variable.  If you call value receiver on a pointer that is `nil`, the program will compile but panic at runtime.

```go
package main

import (
    "fmt"
    "math"
)

type Vertex struct {
    X, Y float64
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := Vertex{3, 4}
    fmt.Println(v.Abs())
}
```

## Pointer and value receivers on methods

You can declare methods with pointer receivers.  This means the receiver type has the literal syntax `*T` for tome type `T` (`T` cannot itself be a pointer such as `*int`)

Methods with pointer receivers can modify the value to which the receiver points Since methods often need to modify their receiver, **pointer receivers** are more common than **value receivers**.

With a value receiver, the method operates on a copy of the original value (same behavior as for any other function argument).  The method must have a pointer receiver to change the value referenced by the receiver.

```go
package main

import "fmt"

type Foo struct {
    A int
    B string
}

// this is a method declaration.
// It looks like a function declaration but has method
// receiver between func and name of method.
// the f here like self in Python
func (f Foo) String() string {
    return fmt.Sprintf("A: %d and B: %s", f.A, f.B)
}

// Method declaration but instead of being declared on Foo its being declared
// on pointer to Foo
// NOTE: if we declared it on Foo a copy would be passed into the double method
// and we would not be able to update the value.
// In general you should declare using value receiver if you are not modifying
// values in struct and reference receiver if you are modifying. 
func (f *Foo) Double() {
    f.A = f.A * 2
}

func main() {
    f := Foo{
        A: 10,
        B: "Hello",
    }
    fmt.Println(f.String())
    f.Double() // you can call a reference or value receiver from a reference that is not a pointer.
    fmt.Println(f.String())
}
```

### Pointers in functions versus pointers in methods

Functions with a pointer argument must take a pointer however methods with a pointer receiver take either a value or a pointer as the receiver when they are called.  This is a convenience; Go intercepts the statement `v.Scale(5)` as `(&v).Scale(5)` (in the example  below) since the `Scale` method has a pointer receiver.

```go
package main

import "fmt"

type Vertex struct {
    X, Y float64
}

func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func main() {
    v := Vertex{3, 4}
    v.Scale(2)
    ScaleFunc(&v, 10)

    p := &Vertex{4, 3}
    p.Scale(3)
    ScaleFunc(p, 8)

    fmt.Println(v, p)
}
```

#### Methods and pointer indirections

Functions that take value arguments must take a value of that specific type.  Methods with value receivers on the other hand, take either a value or a pointer when they are called.  In example below, the method call `p.Abs()` is interpreter as `(*p).Abs()`

```go
package main

import (
    "fmt"
    "math"
)

type Vertex struct {
    X, Y float64
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func AbsFunc(v Vertex) float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := Vertex{3, 4}
    fmt.Println(v.Abs())
    fmt.Println(AbsFunc(v))

    p := &Vertex{4, 3}
    fmt.Println(p.Abs())
    fmt.Println(AbsFunc(*p))
}
```

#### Choosing a value or pointer receiver

There are two reasons to use a pointer receiver.  The first is so that the method can modify the value that its receiver points to.  The second is to avoid copying the value on each method call. This can be more efficient if the receiver is a large struct.

In the example below both `Scale` and `Abc` are methods with receiver type `*Vertex`, even though the `Abc` method does not modify the receiver.

In general all methods on a given type should have either value or pointer receivers, but not a mixture of both.

```go
package main

import (
    "fmt"
    "math"
)

type Vertex struct {
    X, Y float64
}

func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := &Vertex{3, 4}
    fmt.Printf("Before scaling: %+v, Abs: %v\n", v, v.Abs())
    v.Scale(5)
    fmt.Printf("After scaling: %+v, Abs: %v\n", v, v.Abs())
}
```

### How embedding interacts with methods

***Note:*** in example below we are using embedding NOT inheritance

```go
package main

import "fmt"

type Foo struct {
    A int
    B string
}

func (f Foo) String() string {
    return fmt.Sprintf("%d fields: A: %d and B: %s", f.FieldCount(), f.A, f.B)
}

func (f *Foo) Double() {
    f.A = f.A * 2
}

func (f *Foo) FieldCount() int {
    fmt.Println("Foo version of FieldCount")
    return 2
}

type Bar struct {
    C   bool
    Foo // Bar has embedded struct of type Foo
}

func (b Bar) fieldCount() int {
    fmt.Println("Bar version of field count")
    return 3
}

func main() {
    f := Foo{
        A: 10,
        B: "Hello",
    }
    b := Bar{
        C:   true,
        Foo: f,
    }
    fmt.Println(b.String())
    b.Double()
    fmt.Println(b.String()) // Field count of Foo runs
}
```

### Methods on concrete (non struct) types

Go allows you to create concrete types other than structs and define methods on these types like you would with structs.

```go
package main

import "fmt"

type myInt int

func (mi myInt) isEven() bool {
    return mi%2 == 0
}

func (mi *myInt) Double() {
    *mi = *mi * 2
}

func main() {
    m := myInt(10)
    fmt.Println(m)
    fmt.Println(m.isEven())
    m.Double()
    fmt.Println(m)
}
```
