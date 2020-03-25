---
layout: post
title: "Terminal UI using ANSI escape codes"
author: "Miko"
---

Recently, I was writing a tiny tool for the terminal that would have a very
simple UI. I wanted to use whole terminal window and display things in small
boxes. Also, I wanted these to update every few seconds.

My tool was about to be written in Go. There are few really great libraries
out there. However, I thought it should be relatively simple to take ANSI
escape codes and code a simple library that I could re-use in other projects
as well.

### Useful terminal UI libraries
Interesting open source libraries out there are 
[tcell](https://github.com/gdamore/tcell) or 
[termui](https://github.com/gizak/termui). 
Check [Awesome Go](https://awesome-go.com/#advanced-console-uis) for more.

### ANSI escape code
You can give your terminal special instructions by printing ANSI escape codes.
For example, you can tell it to clear the window, change background or 
foreground color, make font bold etc. Various terminals support different
subsets of codes. However, the ANSI escape codes should work on every Unix
and Linux systems.

All codes start with `\u001b[` and for example, we have the following codes:

* `\u001b[3A` moves cursor 3 chars left (`C` moves right, `B` moves down,
`D` moves up);
* `\u001b[2J` clears the terminal window;
* `\u001b[31m` sets the foreground color to red;
* `\u001b[41m` sets the background color to red;
* `\u001b[0m` resets the colors back to normal.

List of escape codes can be found on [Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code#Terminal_output_sequences).

In Golang, a sample code that clears the screen and moves the cursor left
1000 chars and 1000 chars up would be:

```
fmt.Fprintf(os.Stdout, "\u001b[2J\u001b[1000A\u001b[1000D")
```

### go-tui requirements
So it seems it's relatively easy to clear the terminal window, move the cursor
around and change colors - just print an ANSI escape code. I used that for my
library. 

What I precisely needed is abilities to:
* create boxes (which I called panes following tmux, one of my
favorite tools) with specified size in either percentage of width/height or
number of characters;
* split any pane vertically or horizontally;
* add some style to a pane like a frame;
* attach a function to any of the panes which could update the pane contents
dynamically, like every second or something;
* re-render the UI when terminal window gets resized;
* define pane minimal width and height so when it cannot fit it will not be
rendered.

Check the screenshot below.

### Code and example usage

Library can be found on [GitHub](https://github.com/gasiordev/go-tui/).

Check sample code and a screenshot below:

```go
package main

import (
    "os"
    "github.com/gasiordev/go-tui"
)

// TUI has onDraw event and a function can be attached to it. onDraw is
// called when TUI is being drawn, eg. when first started or when 
// terminal window is resized.
// getOnTUIDraw returns a func that will later be attached in main().
func getOnTUIDraw(n *NTree) func(*tui.TUI) int {
    // It does nothing actually.
    fn := func(c *tui.TUI) int {
        return 0
    }
    return fn
}

// TUIPane has onDraw event and a function can be attached to it. onDraw
// is called when TUI is being drawn, eg. when first started or when
// terminal window is resized.
// getOnTUIPaneDraw returns a func that will later be attached in main().
func getOnTUIPaneDraw(n *NTree, p *tui.TUIPane) func(*tui.TUIPane) int {
    // Func is defined separate in another struct which is called a Widget.
    // This Widget prints out current time. Check the source for more.
    t := tui.NewTUIWidgetSample()
    t.InitPane(p)
    fn := func(x *tui.TUIPane) int {
        return t.Run(x)
    }
    return fn
}

func main() {
    // Create TUI instance
    myTUI := tui.NewTUI("My Project", "Its description", "Author")
    // Attach func to onDraw event
    myTUI.SetOnDraw(getOnTUIDraw(n))

    // Get main pane which we are going to split
    p0 := myTUI.GetPane()

    // Create new panes by splitting the main pane. Split creates two
    // panes and we have to define size of one of them. If it's the
    // left (vertical) or top (horizontal) one then the value is lower than
    // 0 and if it's right (vertical) or bottom (horizontal) then the value
    // should be highter than 0. It can be a percentage of width/height or
    // number of characters, as it's shown below.
    p01, p02 := p0.SplitVertically(-50, tui.UNIT_PERCENT)
    p021, p022 := p02.SplitVertically(-40, tui.UNIT_CHAR)

    p11, p12 := p01.SplitHorizontally(20, tui.UNIT_CHAR)
    p21, p22 := p021.SplitHorizontally(50, tui.UNIT_PERCENT)
    p31, p32 := p022.SplitHorizontally(-35, tui.UNIT_CHAR)

    // Create style instances which will be attached to certain panes
    s1 := tui.NewTUIPaneStyleFrame()
    s2 := tui.NewTUIPaneStyleMargin()

    // Create custom TUIPaneStyle. Previous ones are predefined and come
    // with the package.
    s3 := &tui.TUIPaneStyle{
        NE: "/", NW: "\\", SE: " ", SW: " ", E: " ", W: " ", N: "_", S: " ",
    }

    // Set pane styles.
    p11.SetStyle(s1)
    p12.SetStyle(s1)
    p21.SetStyle(s2)
    p22.SetStyle(s2)
    p31.SetStyle(s3)
    p32.SetStyle(s1)

    // Attach previously defined func to panes' onDraw event. onDraw
    // handler is called whenever pane is being drawn: on start and
    // on terminal window resize.
    p11.SetOnDraw(getOnTUIPaneDraw(n, p11))
    p12.SetOnDraw(getOnTUIPaneDraw(n, p12))
    p21.SetOnDraw(getOnTUIPaneDraw(n, p21))
    p22.SetOnDraw(getOnTUIPaneDraw(n, p22))
    p31.SetOnDraw(getOnTUIPaneDraw(n, p31))
    p32.SetOnDraw(getOnTUIPaneDraw(n, p32))

    // Attach previously defined func to panes' onIterate event.
    // onIterate handler is called every iteration of TUI's main loop.
    // There is a one second delay between every iteration.
    p11.SetOnIterate(getOnTUIPaneDraw(n, p11))
    p12.SetOnIterate(getOnTUIPaneDraw(n, p12))
    p21.SetOnIterate(getOnTUIPaneDraw(n, p21))
    p22.SetOnIterate(getOnTUIPaneDraw(n, p22))
    p31.SetOnIterate(getOnTUIPaneDraw(n, p31))
    p32.SetOnIterate(getOnTUIPaneDraw(n, p32))

    // Run TUI
    myTUI.Run(os.Stdout, os.Stderr)
}
```

![Sample code](https://raw.githubusercontent.com/gasiordev/go-tui/master/screenshot.png)

