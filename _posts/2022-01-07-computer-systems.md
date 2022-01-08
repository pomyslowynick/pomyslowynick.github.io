---
layout: post
title:  "Computer Systems"
date:   2022-01-07 23:12:12 +0100
categories: hardware assembly operating-systems computer-science 
---

Watching Carnegie Mellon lectures for Introduction to Computer Systems, focused on the chapters on Machine-Level programming to understand the low level details better.

---

`objdump` -d can disassemble a binary to see what assembly instructions it executes.

`(%rcx)` specifies that a register holds a memory address to be dereferenced. It feels so good to understand it after all this time of looking at gdb outputs.

`(%rdx, %rcx, 4)` - scales rcx by 4 (multiplies)

`0x80 (, %rdx,2)` - scales rdx by 2 then adds an offset of 0x80  

`leaq` - load effective address instruction. Can be used like Cs `&` where it computes an address, but C compiler likes to use it for computing arithmetic expressions of the form x + k * y.
So far I understand that it takes some of the displacement and scale arguments to compute the arithmethic result by bit shifting and then stores it in a register. Ends up being faster than the multiplication operation.