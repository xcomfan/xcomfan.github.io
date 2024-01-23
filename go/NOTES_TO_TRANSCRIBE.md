# Go essentials course from SWQ Go Track

In stings `%v` is the formatted value while `%T` is the Type.

`fmt.Printf("x=%v, type of %T\n", x, x)`

There is a strings package `import "strings"`

This packages has a `Fields` method that lets you split a string. `words := strings.Fields(text)` where `text` is a string.

It also has a `ToLower` method. `strings.ToLower(word)`

`error` is a type in Go.  The function below returns an error or `nil` depending on condition.  `fmt.Errorf` creates the `error` for you.

```go
func sqrt(n float64) (float64, error){
    if n < 0{
        return 0.0, fmt.Errorf("sqrt of negative value (%f)", n)
    }

    return math.Sqrt(n), nill 
}
```

The `defer` statement that lets you run a call when your current function exits.  In the below example the call to release will be executed after worker exits.  You can try this out by having acquire and release print stuff and seeing the running order.

```go
func worker() {
    r1, err := acquire("A")
    if err != nil {
        fmt.Println("ERROR:", err)
        return
    }
    defer release(r1)

    r2, err := acquire("B")
    if err != nil {
        fmt.Println("ERROR:", err)
        return
    }
    defer release(r2)
    
    fmt.Println("worker")
}
```

The example below has some interesting stuff.  First it has 2 ways to print a struct with the `%#v` being a way to print it out with keys and values.  Also some use of the time library.

```go
package main

import (
    "fmt"
    "time"
)

type Budget struct {
    CampaignID string
    Balance    float64
    Expires    time.Time
}

func main() {
    b1 := Budget{"Kittens", 22.3, time.Now().Add(7 * 24 * time.Hour)}
    fmt.Println(b1)
    fmt.Printf("%#v\n", b1)
    fmt.Println(b1.CampaignID)
}
```

In Go if you want to have a constructor for a struct you write a function that does all the work a constructor would do and returns a pointer to the struct.  This function should start with the word `New` by convention.

```go
type Budget struct {
    CampaignID string
    Balance float64
    Expires time.Time
}

func NewBudget(campaignID string, balance float64, expires time.Time) (*Budget, error){
    if campaignID == "" {
        return nil, fmt.Errorf("empty campaignID")
    }

    if balance <= 0 {
        return nil, fmt.Errorf("balance must be bigger than 0")
    }

    if expires.Before(time.Now()){
        return nil, fmt.Errorf("bad expiration date")
    }

    b := Budget{
        CampaignID: campaignID,
        Balance: balance,
        Expires: expires
    }

    return &b, nil
}

func main(){
    expire := time.Now().Add(& * 24 * time.Hour)

    b1, err := NewBudget("puppies", 32.2, expire)
    if err != nil {
        fmt.Println("ERROR": err)
    } else {
        fmt.Printf("%#v\n", b1)
    }
}
```

A good way to think about interfaces.  Think of shapes circle, triangle and square. If we use interfaces to give each an area method and call the interface Shape, we can now create a slice of shapes.  Even though they are different structs because they implement the same interface we can treat them as the same type and stick them into a slice.

In Go IO reader is an interface and using this same interface you can work with files, sockets compressed files etc.

Generics is a feature added to go 1.18 that lets you use a set of types as an interface.

```go
type Ordered interface {
    int | float64 | string
}

func min[T Ordered](values []T) (T, error){
    if len(values) == 0 {
        var zero T
        return zero, fmt.Error("min of empty slice")
    }

    m := values[0]
    for _, v := range values[1:]{
        if v < m {
            m = v
        }
    }

    return m, nil
}

func main(){
    fmt.Println(min([]float64{2, 1, 3}))
    fmt.Println(min[]string{"B", "A", "C"})
}
```

`constraints.Ordered` in Go is a type constraint that permits only ordered types (types that support <, <=, >= > operators).

```go
func myFunctionT constraints.Ordered T {
    if a < b {
        return b
    }
    return a
}
```

## Error handling

There is a commonly used package called pkg errors.  This is a drop in replacement to the built in pkg/errors package.

Below is a good example of how to open up a config file and handle the possibility of it being missing as well as some basic logging.  The third party package they are importint did not work for me but I did ot try too hard to get it running.  Still the flow is useful to transcribe.

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

You can have your code create a panic if needed by calling the `panic("oops")` function.  This is however not considered idiomatic Go code so avoid doing this.

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

## Concurrency

In the example below we have a serial and concurrent version of calling a list of URLs and timing how long they run.

In the concurrent version we are using a wait group to track concurrent execution.  We add 1 to wait group then call then we create a function and run it in a separate goroutine with the `go` keyword.  The function we create signals the wait group that it is complete with the `wg.Done()` call and we wait for all the functions being called to complete with `wg.Done()`

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
    "time"
)

func returnType(url string) {
    resp, err := http.Get(url)
    if err != nil {
        fmt.Printf("Error: %s\n", err)
        return
    }

    defer resp.Body.Close()
    ctype := resp.Header.Get("content-type")
    fmt.Printf("%s -> %s\n", url, ctype)

}

func siteSerial(urls []string) {
    for _, url := range urls {
        returnType(url)
    }
}

func sitesConcurrent(urls []string) {
    var wg sync.WaitGroup
    for _, url := range urls {
        wg.Add(1)
        go func(url string) {
            returnType(url)
            wg.Done()
        }(url)
    }
    wg.Wait()
}

func main() {
    urls := []string{
        "https://golang.org",
        "https://api.github.com",
        "https://httpbin.org/ip",
    }

    start := time.Now()
    siteSerial(urls)
    fmt.Println(time.Since(start))

    start = time.Now()
    sitesConcurrent(urls)
    fmt.Println(time.Since(start))
}
```

The preferred way to communicate between Goroutines is by using channels.  Thing of a channel as a pipe that can send a specific type such as an int or a string.  We can send and receive on a pipe, and each will block if there is not a recipient at the other end.

Below is a basic example of sending to a channel and receiving the value.  Note that send and receive without the Goroutine you would get a deadlock.

Note in the loop example below if we close the channel after the loop that allows us to use `range` to iterate over the channel.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int)

    go func() {
        // Send number to the channel
        ch <- 42
    }()

    val := <-ch // receive the value
    fmt.Printf("got %d form channel\n", val)

    const count = 3

    go func() {
        for i := 0; i < count; i++ {
            fmt.Printf("sending %d\n", i)
            ch <- i
            time.Sleep(time.Second)
        }
        close(ch)
    }()

    for i := range ch {
        fmt.Printf("Received %d\n", i)
    }
}
```

Select lets you work with multiple channels.

Below is a basic example of select.

```go
package main

import "fmt"

func main() {
    ch1, ch2 := make(chan int), make(chan int)

    go func() {
        ch1 <- 42
    }()

    select {
    case val := <-ch1:
        fmt.Printf("got %d from ch1\n", val)
    case val := <-ch2:
        fmt.Printf("got %d form ch2\n", val)
    }
}
```

One use case of select statements is timeouts.  Below is an xample of using `select`.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    out := make(chan float64)

    go func() {
        time.Sleep(100 * time.Millisecond)
        out <- 3.14
    }()

    select {
    case val := <-out:
        fmt.Printf("got %f\n", val)
    case <-time.After(20 * time.Millisecond):
        fmt.Println("timeout!!!")
    }
}
```

The `%` placeholders in a string are called verbs, and the `%T` will print the variable type.

In Go like in Java characters are in single quotes `'a'` and strings in double quotes `"a string"`

## Get user input

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {

    reader := bufio.NewReader(os.Stdin)
    fmt.Print("Enter text: ")
    input, _ := reader.ReadString('\n')
    fmt.Println("You entered:", input)
}
```

## Convert strings to other types

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strconv"
    "strings"
)

func main() {

    reader := bufio.NewReader(os.Stdin)
    fmt.Print("Enter text: ")
    input, _ := reader.ReadString('\n')
    fmt.Println("You entered:", input)
    fmt.Print("Enter a number: ")
    numInput, _ := reader.ReadString('\n')
    aFloat, err := strconv.ParseFloat(strings.TrimSpace(numInput), 64)
    if err != nil {
        fmt.Println(err)
    } else {
    fmt.Println("Value of number: ", aFloat)
    }
}
```

## Working with time

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    n := time.Now()
    fmt.Println("I am running this code at: ", n)

    t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
    fmt.Println("Go launched at ", t)

    fmt.Println(t.Format(time.ANSIC))

    parsedTime, _ := time.Parse(time.ANSIC, "Tue Nov 10 23:00:00 2009")
    fmt.Printf("The type of parseTime is %T\n", parsedTime)
}

```

## Parsing Floats

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    f1, f2, f3 := 23.5, 65.1, 76.3
    floatSum := f1 + f2 + f3
    // prints x.yyyyyyyyy due to binary storage
    fmt.Println("Float sum:", floatSum)

    // *100 / 100 needed because by default math.Round will give integer
    //This is the safest and easiest way in Go to round numbers adn keep the decimal portion.
    // the *100 / 100 will give you 2 decimals of precisions (the number of 0s in 100) 
    // for 3 use 1000
    floatSum = math.Round(floatSum*100) / 100
    fmt.Println("The sum is now", floatSum)
}
```

## Memory management

### Make() vs new()

`new()` Allocates but does not initialize memory. Returns a memory address but zeroed out storage. 

`make()` Allocate and initialized memory. Allocates non-zeroed storage and returns a memory address.

`*` means its a pointer `&` dereferences the pointer.

## Complex types

To print a struct for debbugging purposes use `%+v` for example `fmt.Printf("%+v\n", poodle)` where `poodle` is an instance of a struct.

