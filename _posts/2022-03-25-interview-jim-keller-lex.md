---
layout: post
title: "Notes from Jim Keller Interview "
date: 2022-03-25 19:52:10 +0100
categories: notes interviews videos hardware
---

Instructions set are stable for relative amount of time.
Instruction set - how do you implement basic operations.

> 90% of execution runs on 20 opcodes. Those are stable.

Intel Architecture has been around for 25 years.

Modern CPU fetches a lot of instructions, Jim gives a figure of about 500, then it builds a graph of data dependencies between them and then executes independent units, micrographs.
Market for simple, clean computers is zero.

Computer has a bunch of "book keeping tables" which allow it to modify the dependency graph and execute instructions efficiently.

GPUs - they fetch a bunch of instructions dependent instructions and executes those dependable instructions in parallel and out of order because they fit the same "one serial narrative". 
Given parallelism, we don't really care about the order in which operations are done for the pixels calculation, the problem is inherintly parallel. Just like web search or quotation indexing in Barroso book.

CPU level parellelism can in theory introduce 10x speed up, just like VTune score says. The perfect score is to execute one instruction per 0.25 cycle. The sequential execution can compute about 1 instruction per 3 cycles.

Found parallelism and predictability of operation are important factors. 
On average a branch instruction happens every 6 instructions.

Accuracy of branhc prediction using the last recorded sate is about 85%.
Branch prediction using the traversed path in the graph is more accurate, since kinda natural.
Taking history into account increases the branch prediction accuracy, according to Jim the model looks a bit like a neural network. 

Some work done during branch misprediction can be recovered.

> Is there room for fuzziness in the computer results?

The definition of language is that you will get the same answer for the same code.
While the answers remain deterministic, everything that is going into the modern calculations is noisy.
You get the deterministic answer, even though the process underneath it is very nondeterministic. You always arrive at the same answer, despite the process being inherintely indetereministic, just like Lex I think this is amazing.

I like how Jim describes the ability to know when to execute a recipe and when to develop understanding as crucial decision making element.

> If you wanna do real progress on computer architecture, every 3-5 years you should start from scratch

That is about the most counter intuitive thing I've heard, but I think the logic behind it is to rethink the fundamentals more so than throw away the designs developed throughout the years.

New computer/cpu etc. might be faster among a wide distribution of metrics, but there always might be individual factors which it will perform worse on. And big companies selling to lots of customers care about all those metrics, as some of the customers will definitely do.

Moore's law in it's simplest for is the statement that every 2-3 years 2x more transistors can be squeezed onto a die. The general understanding that it improves the performance two fold is mistaken, the improvements come from a plethora of different mechanisms implemented on the hardware level.
Jim likens the graph of performance improvement, when broken down into individual componenets, to a graph of diminishing returns for each individual component.

> Modern transistors are on a scale of 1 / (1000 x 1000 x 1000) atoms. At this level quantum effects take effect, so the quantum computing is about exploiting the quantum mechanics at this level.

Keller seems to be more interested in the human factors as constraints to developing more complex processors, he says that people working within teams of 10 to 100 are the hard constraint on the progress that can be made.

> Simply using faster computers to build bigger computers doesn't work

There is mathematical hierarchy to parallelism of operations, but also to the amount of work that can be processed in a single unit, single equation, vector, matrix. This is a very interesting observation from a purely mathematical view of CS.

Data in AI has become so big that from the mathematical perspective it's a topology problem, a problem of describing an immense shape.

> Difference between a given search space and found search space

> The space mathematics can describe is infinite. # Needs a proof


> The difference in quantity is a difference in kind

[Emergence](https://en.wikipedia.org/wiki/Emergence) is a systems theory which tackles this viewpoint touched upon in the later discussions.

Nano wires - one of the newest developments in the metallurgy aspect of transistors work.

Analogy of performance hierarchy by Keller:
- Well designed general purpose CPU has value of 1
- GPU with it's granted parallelism has value of 5
- Accelerator built on top of that is about 2-3 times faster than a GPU, it can be granted that mathematical operations will always be of a certain kind and as such they can optimized. Operations are a subset of mathematical possibilities.