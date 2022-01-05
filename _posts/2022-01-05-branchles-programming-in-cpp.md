---
layout: post
title:  "Branchless Programming in C++ - Fedor Pikus - CppCon 2021"
date:   2022-01-05 21:20:45 +0100
categories: cpp hardware performance systems
---


[Link to the presentation](https://www.youtube.com/watch?v=g-WPhYREFjk)

--- 

Data dependencies - first time I've heard the term, but I really like it. Most instructions have data dependecies on other instructions.
If branches are "code" dependencies. CPU doesn't know which instructions will be executed until it processes the result.

Pipelining - extension of ability to execute myltiple operations at once. We can identify streams of data dependencies which can be execute in parallel.
In the example given we have a two sums and then the product of them computed within a loop. We can execute the addition for current loop iteration and multiplication of the previous two sums in one "set" of instructions to utilize more of CPU. 

The goal of pipelining is to have multiple streams of instructions running in parallel, thus CPU is utilized to it's full potential.
Incrementation is usually "embedded" into an instruction.

Branches are bane of all the pipelines, CPU must wait for the result of the computation to establish a pipeline.
x86 cmove is an instruction which does not add additional overhead to branching, if compiler has replaced a branch with in, and in limited cases it can do that, we can safely ignore it for the performance purposes.

Branch predictors are trying to cater for that.
Hardware loop unrolling - some loops can be representated as a series of instructions which can be processed immediately, instead of waiting for the result of a previous iteration.

Are we really surprised? Turns out eax, ebx etc. are not real physical registers, they are another layer of abstraction over the hardware. Real register names are different. I don't even know why it surprised me :) 
There are a lot more physical registers than logical ones, eax could be used twice within the same loop and be two different registers. It's called register aliasing. 

In branch mispredictions error might happen (null dereferencing for example) which then have to be suspended until we take the correct branch.
What is even more interesing is that speculative writes cannot be undone, so CPU has write buffers which (as claimed by Fedor) were designed for that exact purpose.

> I always pick up some interesting tools and tricks from the good talks I've seen, here the guy is using [vimdiff](https://www.tutorialspoint.com/vim/vim_diff.htm) which is a nice handy way to visualize a file diff in vim.

Only try to fix mispredictions if you have profiled the program and verified their existence.

Bitwise or can be more efficient than a short circuited C version of OR, if we know that the branch will be always true/false, but each condition evaluates to true at random.

Depending on goals we could optimize for the most number of computations done, like in an example of a commercial server, or doing least amount of work not to drain the battery, like in the case of a smartphone.