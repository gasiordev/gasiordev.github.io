---
layout: post
title: "Key press event in Go"
author: "Mikolaj Gasior"
---

To handle key press event in Go CLI application we have to change STTY flags.

By default we have to send new line to get our input send to the terminal. We
can change that by running `stty cbreak min 1`.
Also, we might want to not have our input echoed back. Running `ssty -echo`
mutes the thing.

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

We always have to revert terminal flags to the way they were before running
application. Command `stty sane` should do the trick. It will fix "broken"
terminal. Note there are other more apropriate ways to do that.

## Links

Check some of the following links for more info:
* [https://www.computerhope.com/unix/ustty.htm](https://www.computerhope.com/unix/ustty.htm)
* [https://dev.to/napicella/linux-terminals-tty-pty-and-shell-192e](https://dev.to/napicella/linux-terminals-tty-pty-and-shell-192e)
