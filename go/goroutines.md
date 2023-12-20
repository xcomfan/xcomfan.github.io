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

