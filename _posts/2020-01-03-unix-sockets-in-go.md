---
layout: post
title: "Unix sockets in Go"
author: "Miko"
---

In one of the application I had to make use of Unix sockets so I thought it
could be useful to share a quick working example of how easy it is to code
a simple server and a client. 

Check the code below. Socket path in the code is `/tmp/app.sock` - you might
want to change it in case it does not work. 
Run both scripts on separate terminals with eg. on one terminal type 
`go run server.go` and then `go run client.go` on the other.
If you get a problem with binding to a socket then remove your socket file.
It's just a sample code.

## server.go
```
package main

import (
    "net"
    "log"
)

func readFromSock(c net.Conn) {
    for {
        buf := make([]byte, 512)
        cnt, err := c.Read(buf)
        if err != nil {
            return
        }

        data := buf[0:cnt]
        println("Data received from Client:", string(data))
        _, err = c.Write([]byte("Thanks for \"" + string(data) + "\""))
        if err != nil {
            log.Fatal("Write to client error: ", err)
        }
    }
}

func main() {
    l, err := net.Listen("unix", "/tmp/app.sock")
    if err != nil {
        log.Fatal("Listen error:", err)
    }

    for {
        fd, err := l.Accept()
        if err != nil {
            log.Fatal("Accept error:", err)
        }

        go readFromSock(fd)
    }
}
```

## client.go
```
package main

import (
    "net"
    "io"
    "log"
    "time"
)

func readFromSock(r io.Reader) {
    buf := make([]byte, 1024)
    for {
        n, err := r.Read(buf)
        if err != nil {
            return
        }
        println("Data received from Server:", string(buf[0:n]))
    }
}

func main() {
    c, err := net.Dial("unix", "/tmp/app.sock")
    if err != nil {
        log.Fatal("Dial error: ", err)
    }
    defer c.Close()

    go readFromSock(c)

    for {
        _, err := c.Write([]byte("Hello there!"))
        if err != nil {
            log.Fatal("Write error:", err)
            break
        }
        time.Sleep(time.Millisecond * time.Duration(2000))
    }
}
```
