---
layout: page
title: "Getting user input "
permalink: /go/user_input
---

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
