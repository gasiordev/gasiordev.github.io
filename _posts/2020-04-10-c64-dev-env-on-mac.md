---
layout: post
title: "Commodore 64 development environment on Mac"
author: "Mikolaj Gasior"
---

To make a simple C64 devkit on your Mac computer you need GNU Make, C64
emulator and assembler compilator. I have used VICE for the emulation and
64tass for assembly.

## Install VICE
You can get VICE emulator 
[here](https://vice-emu.sourceforge.io/index.html#download).

Download latest one for MacOS (I use the SDL version) and place it somewhere,
eg. in `/Applications`. In my case it's `/Applications/vice-sdl2-3.4-r37482`.

Add `bin` directory to `PATH` variable. If you use bash, you can put it at the
end of `~/.bash_profile`:

```
export PATH=$PATH:/Applications/vice-sdl2-3.4-r37482/bin
```

Reload your console and type `x64sc -help` to check if emulator binary is 
globally available.

## Assembler
Download latest release source code from 
[GitHub](https://github.com/irmen/64tass/releases) and unpack it somewhere.

Go the created directory and run `make` to compile the binary. Check if file
called `64tass` has been created by running `./64tass -V`. If so, copy the
file to `/usr/local/bin` and make it executable 
`chmod +x /usr/local/bin/64tass`.

## Makefile
Go to the directory where you want to work on your project and create the
following `Makefile` to make life slightly easier:

```
X64=x64sc
ASM=64tass

all: demo.prg
    x64sc -autostartprgmode 1 -chdir `pwd` -autostart-warp +truedrive +cart $<

demo.prg: demo.s demo.sid
    64tass -C -a -B -i $< -o $@

.PHONY: all clean
clean:
    rm -f demo.prg
```

Remember that `Makefile` should use tabs for indentation!

Replace `demo` with your name.

## Sample project
Let's run some assembler code. Create `demo.s` and put the following
contents into it:

```
           * = $0801

           .word (+), 2016
           .null $9e, format("%d", start)
+          .word 0

start      lda #$00
           sta $d020
           sta $d021
           tax
           lda #$00
clrscr
           sta $0400,x
           sta $0500,x
           sta $0600,x
           sta $0700,x
           sta $2000,x
           dex
           bne clrscr
           lda #$018
           sta $d018
mainloop
           lda $d012
           cmp #$fe
           bne mainloop
           ldx counter
           inx
           cpx #88
           bne changechr
           ldx #$00
changechr
           stx counter
           lda $2000,x
           eor #$ff
           sta $2000,x

           jmp mainloop
counter
           .byte 8
```

Now type `make` to run it.

## Links
There's plenty of good resources regarding coding for Commodore 64. A great
starting point is [CodeBase64](https://www.codebase64.org).

VICE Emulator manual can be found [here](https://vice-emu.sourceforge.io/vice_toc.html).
64tass is very well documented on its [site](http://tass64.sourceforge.net/).

