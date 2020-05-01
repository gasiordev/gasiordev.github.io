---
layout: post
title: "Key press event in Go"
author: "Mikolaj Gasior"
---

To handle key press event in Go in our CLI application we have to change flags
for the TTY we use. 

By default we have to press enter (new line) to get our input send. We can
change that by running `stty cbreak min 1`.
Also, we might want to not have our input echoed back. Run `ssty -echo` to get
that set.

## Sample Go code
Let's have a look how we can use that in Go code.

```
package main

import (
    "os"
    "os/exec"
    "fmt"
    "log"
)

func startStdioLoop() {
    var b []byte = make([]byte, 1)
    for {
        os.Stdin.Read(b)
        fmt.Printf("Keypress: " + string(b) + "\n")
        if string(b) == "q" {
            cmd := exec.Command("stty", "-echo")
            cmd.Stdin = os.Stdin
            _ cmd.Run()
            os.Exit(0)
        }
    }
}

func main() {
    cmd1 := exec.Command("stty", "cbreak", "min", "1")
    cmd1.Stdin = os.Stdin
    err := cmd1.Run()
    if err != nil {
        log.Fatal(err)
    }
    cmd2 := exec.Command("stty", "-echo")
    cmd2.Stdin = os.Stdin
    err = cmd2.Run()
    if err != nil {
        log.Fatal(err)
    }
    
    done := make(chan bool)
    go startStdioLoop()
    <-done
}
```

You have to revert your terminal flags to the way they were before running your
application. You can use `stty sane` command. It will try to fix your "broken"
terminal (look at the code above again).

## Links

Check some of the following links for more info:
* https://www.computerhope.com/unix/ustty.htm(https://www.computerhope.com/unix/ustty.htm)
* https://dev.to/napicella/linux-terminals-tty-pty-and-shell-192e(https://dev.to/napicella/linux-terminals-tty-pty-and-shell-192e)
