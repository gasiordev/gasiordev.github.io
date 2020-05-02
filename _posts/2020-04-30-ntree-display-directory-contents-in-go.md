---
layout: post
title: "ntree, display directory contents in Go"
author: "Mikolaj Gasior"
---

In last months, Go has become my primary programming language for any scripting
or creating apps that "glue" different tools.

One day I sat down to write a tool that could display contents of a directory.
Something that I could run in a separate pane in tmux and have refreshed
automatically when I change directory while working on other panes (usually
terminals). I ended up doing a tiny program called ntree.

Check the simple screenshot (in real life I use much bigger terminal window).

![Sample code](https://raw.githubusercontent.com/gasiordev/ntree/master/ntree.png)

The source code can be found on my [GitHub](https://github.com/gasiordev/ntree).
I used some of libraries I created earlier: 
[go-cli](https://github.com/gasiordev/go-cli) and
[go-tui](https://github.com/gasiordev/go-tui).

Maybe you can find it useful.

