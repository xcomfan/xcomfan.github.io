---
layout: page
title: "Goroutines"
permalink: /go/goroutines
---

[comment]: <> (TODO: This sections is just raw notes from the class.  I need to review and reformat.  Also should break out the channel and select sections)

## Goroutines

Goroutines use CSP which stands for communicating sequential processes and its what concurrency in Go is based on.

Goroutines are lightweight processes managed by the Go runtime.

The Go runtime schedules goroutines across threads automatically.

Goroutines are faster to create, more efficient with memory, and faster to switch.

Can have tens of thousands of goroutines in a single process

```go
package main

import (
    "fmt"
    "sync"
)

func runMe() {
    fmt.Println("Hello from a goroutine")
}

func main() {
    // You need to use a wait group to wait for Goroutines to finish
    var wg sync.WaitGroup
    // Tell wait group we are adding 1 go routine to run
    wg.Add(1)
    // Launch a closure as a go routine and in the closure call runMe and Done
    // method of wait group.
    go func() {
        runMe()
        wg.Done()
    }()

    // Outside of closure back in main we wait for wg
    // will pause main till wg count reaches 0.
    wg.Wait()
}
```

Note that in the example above the runMe method does not need to do anything to be concurrent.  Concurrency logic should be separate from your business logic.

Now lets look at an example of passing data to a Go routine

```go
package main

import (
    "fmt"
    "sync"
)

func runMe(name string) {
    fmt.Println("Hello to", name, "from a goroutine")
}

func main() {
    // You need to use a wait group to wait for Goroutines to finish
    var wg sync.WaitGroup
    // Tell wait group we are adding 1 go routine to run
    wg.Add(1)
    // Launch a closure as a go routine and in the closure call runMe and Done
    // method of wait group.
    go func(name string) {
        runMe(name)
        wg.Done()
    }("Bob")

    // Outside of closure back in main we wait for wg
    // will pause main till wg count reaches 0.
    wg.Wait()
}
```

Note that you can access the variables from the outside as you are doing with wg, but if variable is changed before go routine runs you own't get value that you expect.

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            fmt.Println(i) // if you do this all go routines will share the same i
            // and you will get unpredictable results.
            wg.Done()
            }()
    }

    wg.Wait()
}
```

correct way is

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(localI int) {
            fmt.Println(localI) // if you do this all go routines will share the same i
            // and you will get unpredictable results.
            wg.Done()
        }(i)
    }

    wg.Wait()
}
```

# Notes from Learn Go in 3 hours class

**NOTE:** Go does not consider a zero value to be false.



## Channels

Channels are used to send data to and from a Goroutine while it is running.

One or more Goroutines write to a channel and one or more Goroutines read from the same channel.

Data on channels is typed.

By default, channel reads and writes are synchronous.

Putting a value in a channel is just like passing a value to a function.  If its not a reference type a copy is made and sent to the reading Goroutine

Be careful when passing reference types over a channel.

```go
package main

import (
    "fmt"
)

func main() {
    // declare our in and out channels
    in := make(chan string)
    out := make(chan string)
    // launch our goroutine
    go func() {
        // read from the in channel
        // <- is the receive operator.
        name := <-in
        out <- fmt.Sprintf("Hello, " + name)
    }()
    // write string to the in channel.
    in <- "Bob"
    // close the channel
    close(in)
    // read from out
    // notice that we did not need a wait group
    // this is because the main function is waiting to read from the out channel
    // which has nothing on it till the goroutine completes
    // if you don't want to wait for a channel to be read after we write to it use
    // buffer channel.
    message := <-out
    fmt.Println(message)
}

```

Buffer channels are not infinite and when full Goroutines will pause till the channel is read from.

```go
package main

import "fmt"

func main() {
    // create a buffer channel with room for 10 items of type int.
    out := make(chan int, 10)
    // loop passing each value to a go routine
    for i := 0; i < 10; i++ {
        go func(localI int) {
            out <- localI * 2
        }(i)
    }
    var result []int
    // read from the out channel and append it to our results
    for i := 0; i < 10; i++ {
        val := <-out
        result = append(result, val)
    }
    fmt.Println(result)
}
```

This is one of the uses of buffer channel gather data from a number of workers when you know how many to expect.  

In the example below we are going to get a deadlock when it runs.

```go
package main

import "fmt"

func main() {
    in := make(chan int)
    out := make(chan int)

    go func() {
        // for loop runs forever reading from in and writing the double to out
        for {
            i := <-in
            out <- i * 2
        }
    }()

    in <- 1
    in <- 2
    o1 := <-out
    o2 := <-out

    fmt.Println(o1, o2)
}
```

You need to make sure that a Goroutine is not blocked reading from or writing to a channel.  Fix for the deadlock above is to write read write read not write, write, read read.

```go
package main

import "fmt"

func main() {
    in := make(chan int)
    out := make(chan int)

    go func() {
        // for loop runs forever reading from in and writing the double to out
        for {
            i := <-in
            out <- i * 2
        }
    }()

    in <- 1
    o1 := <-out
    in <- 2
    o2 := <-out

    fmt.Println(o1, o2)
}
```

Closing channels

```go
package main

import "fmt"

func main() {
    in := make(chan int, 10)
    out := make(chan int)

    for i := 0; i < 10; i++ {
        in <- i
    }
    // closing channel means no more values written to it, but contents are not wiped out
    // any values in buffer still available to be read.
    // reading from closed channel with no data remaining will give the 0 value.
    close(in)

    go func() {
        for {
        // ok is for us to be able to tell diff between closed channel and just 0
        i, ok := <-in
            if !ok {
                close(out)
                break
            }
            out <- i * 2
        }
    }()

    // you can have a for range loop with channels
    // the v will be the latest value that was written to the channel.
    // when channel closed and there are no more values to be read the for loop
    // will exit.
    for v := range out {
        fmt.Println(v)
    }
}

```

Be sure to not write to a closed channel or to try to close a closed channel or you will get a panic.

The zero value for a channel is `nil`.  Writing to a `nil` channel makes your go routine hang forever.  Closing a `nil` channel will cause a panic.

| Op | unbuffered | buffered | nil | closed |
| -- | ---------- | -------- | --- | ------ |
| read | Pause until something is written | Pause if buffer is empty | Hang forever | Return immediately w/zero value (use comma-ok to see if it is closed) |
| write | Pause until something is read | Pause only if buffer full | Hang forever | PANIC |
| close | Works | Works | PANIC | PANIC |



## Select

What if yo had multiple channels.  You would want to read the ready channels and skip the ones that are blocked.  Also what if you want to only write to a channel that is not blocked.  This is possible with select.

## Goroutine notes from tour of Go

A **goroutine** is a lightweight thread managed by the Go runtime.  `go f(x, y, z)` starts a new goroutine running `f(x, y, z)`.  The evaluation of `f, x, y` and `z` happens in the current goroutine and the execution of `f` happens in the new goroutine.

Goroutines run in the same address space, so access to shared memory must be synchronized.

```go
package main

import (
    "fmt"
    "time"
)

func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    go say("world")
    say("hello")
}
```

### Channels again

Channels are a typed conduit through which you can send and receive values with the channel operator `<-`.

```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```

(The data flows in the direction of the arrow)

Like maps and slices, channels must be created before use. `ch := make(chan int)`

By default, sends and receives block until the other side is ready.  This allows goroutines to synchronize without explicit locks or condition variables.  The example code sums the numbers in a slice, distributing the work between two goroutines.  Once both goroutines have completed their computation, it calculate the final result.

```go
package main

import "fmt"

func sum(s []int, c chan int) {
    sum := 0
    for _, v := range s {
        sum += v
    }
    c <- sum // send sum to c
}

func main() {
    s := []int{7, 2, 8, -9, 4, 0}

    c := make(chan int)
    go sum(s[:len(s)/2], c)
    go sum(s[len(s)/2:], c)
    x, y := <-c, <-c // receive from c

    fmt.Println(x, y, x+y)
}
```

### Buffered Channels

Channels can be buffered.  Provide the buffer length as the second argument to `make` to initialize a buffered channel.  Sends to a buffered channel block only when the buffer is full.  Receives block when the buffer is empty.  

`ch := make(chan int, 100)`

### Range and close

A sender can close a channel to indicate that no more values will be send.  Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression.

`v, ok := <- ch`

`ok` is `false` if there are not more values to receive and the channel is closed.  The loop `for i := range c` receives values form the channel repeatedly until it is closed.  ***Note:*** Only the sender should close the channel, never the receiver.  Sending on a closed channel will cause a panic.  ***Note:*** Channels aren't like files; you don't usually need to close them.  Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a `range` loop.

```go
package main

import (
    "fmt"
)

func fibonacci(n int, c chan int) {
    x, y := 0, 1
    for i := 0; i < n; i++ {
        c <- x
        x, y = y, x+y
    }
    close(c)
}

func main() {
    c := make(chan int, 10)
    go fibonacci(cap(c), c)
    for i := range c {
        fmt.Println(i)
    }
}
```

### Select from go tour

The `select` statement lets a goroutine wait on multiple communication operations.  A `select` blocks until one of its cases can run, then it executes that case.  It chooses one at random if multiple are ready.

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}

func main() {
    c := make(chan int)
    quit := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)
        }
        quit <- 0
    }()
    fibonacci(c, quit)
}
```

#### Default Selection

The `default` case in a `select` is run if no other case is ready.  Use a `default` case to try a send or receive without blocking

```go
select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(100 * time.Millisecond)
    boom := time.After(500 * time.Millisecond)
    for {
        select {
        case <-tick:
            fmt.Println("tick.")
        case <-boom:
            fmt.Println("BOOM!")
            return
        default:
            fmt.Println("    .")
            time.Sleep(50 * time.Millisecond)
        }
    }
}
```

### sync.Mutex

We've seen how channels are great for communication among goroutines.  But what if we don't need communication? What if we just want to make sure only one goroutine can access a variable at a time to avoid conflicts.  This concepts is called *mutual exclusion* and the conventional name for a data structure that promotes it is *mutex*.  Go's standard library provides mutual exclusion with `sync.Mutex` and its two methods `Lock` and `Unlock`.  We can also `defer` to ensure the mutex will be unlocked as in the `Value` method

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()
    // Lock so only one goroutine at a time can access the map c.v.
    c.v[key]++
    c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
    c.mu.Lock()
    // Lock so only one goroutine at a time can access the map c.v.
    defer c.mu.Unlock()
    return c.v[key]
}

func main() {
    c := SafeCounter{v: make(map[string]int)}
    for i := 0; i < 1000; i++ {
        go c.Inc("somekey")
    }

    time.Sleep(time.Second)
    fmt.Println(c.Value("somekey"))
}
```
