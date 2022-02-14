---
layout: post
title:  "MIT 6.172 Performance Engineering of Software Systems, Fall 2018"
date:   2022-02-04 23:16:12 +0100
categories: course notes performance hardware benchmarking profiling 
---

## 1. Introduction and Matrix Multiplication
The static current within the CPUs became a bigger problem than the dynamic one, as the leakage currents became a serious issue. This affected the clock frequencies which couldn't be scaled anymore past some point.

Since 2004/5 parallelism in the form of multicore processors was introduced as a way to improve the performance. Clock speed has remained pretty much constant since.

https://en.wikipedia.org/wiki/Dennard_scaling

https://en.wikipedia.org/wiki/Die_(integrated_circuit)

### Matrix multiplication problem

Comparison of Python, Java and C speed

https://docs.microsoft.com/en-us/archive/msdn-magazine/2015/february/compilers-what-every-programmer-should-know-about-compiler-optimizations

https://stackoverflow.com/questions/28998690/code-reordering-due-to-optimization

clang -Rpass=vector -> will produce a vectorization report telling you which instructions have been vectorized. Many machines don't support the newest set of vector instructions so compiler uses the convervatively.

Floating point operations are not associative, A * B * C can have a different result depending on the order of operations.

`-ffast-math` - might be required to allow some vectorization because of the floating point arithmetic restrictions.

[Intrinsics guide](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=store&ig_expand=6907,6906,6898)

## 2. Bentley Rules for Optimizing Work

* Packing - fit more than one data value in a machine word, related to the idea of encoding when we convert data values into a representation that requires fewer bits. 

``` C
typedef struct {
    int year: 13;
    int month: 4;
    int day: 5;
} date_t;
```

You can define the amount of bits each field within the struct should take after the colon, like in the example above when it adds up to 22 bits which have a structure that can be easily used to retrieve the data.

Sometimes unpacking and decoding can result in better performance, depending on the access patterns.

* Augmentation - adding to a data structure information which makes common operations more efficient.

An example could be adding a tail pointer to a linked list so that append operations can be performed much quicker. (that is without traversing the whole list)

* Precomputation - perform calculations in advance, basically populate a table of values which would have to be recomputed. Libe fibonacci numbers, binomials etc.

For binomial you can precompute a table of coefficients .
Pascal's triangle is the binomial coefficient table.

* Compile-Time initialization - store values of constants during compilaton, store your table of coefficients as a table in the source file.

We can create those large static tables through metaprogramming, in the example we use printf statements to construct this table. (I assume we will use a macro eventually)

You could create a Makefile which pipes the output of a program generating a program. Any scripting language could generate those structures through the meta programming.

* Caching - store results that have been accessed recently so you don't need to recompute them.

sqrt is an expensive operation on modern machines. In the lecture a software level cache is shown, we store the value in variables. I believe this would be called memoizing in the context of functional languages?

It makes me realize how there are multiple levels at which caching can be done, software, hardware.

* Sparsity - avoid storing and computing on zero elements of input.

Compressed Sparse Row - Keep rows and indexes of the matrix fields which hold non-zero values. Basically augment the matrix to hold offsets of fields holding values.

* Constant folding and propagation - evaluate constant expressions and substitute the results into further expressions. With sufficiently high optimization level compiler should do it on it's own. It's good to know about it, but most of the time compiler will take care of it.

* Common subexpression elimination - avoid computing the same expression multiple times by evaluating the expression once and storing the result for later use.

Compiler will detect some of these, but not all.

* Algebaic identities - replace expensive algebraic expressions with equivalents that take less work

[Fast path optimization?](https://en.wikipedia.org/wiki/Fast_path)

* Short-circuiting - stop evaluating expressions as soon as the answer is known.

Based on the frequency of the short-circuit condition it mgiht actually slow down the computation.

* Ordering tests - tests which end up being successful more often should be put before those that succeed rarely. Same logic applies to tests which are less computationally expensive, we should perform them earlier. This is application specific.

* Creating a fast path - check for a condition before executing more expensive computation. If some condition can be identified as a prerequisite then it can serve as a "fast path" which avoids computation. 

* Combine tests - replace a sequence of tests with one test or a switch. 

Didn't think of it before, but multiple conditions can be broken into a range of values within a single number, in lecture it is achieved by moving bits a certain amount for each of the variables.

### Loops

* Hoisting - avoid recomputing loop-invariant code each time through the body of a loop.

Move a constant expression outside of the loop. Sometimes compiler will be able to figure it out.

* Sentinels - special dummy values placed in data structure to simplify the logic of the boundary conditions.

1:04:00 this is broken. Signed integer overflow is undefined behavior. Some optimizing compilers could remove the (sum < A[i]) check, because they’re allowed to assume undefined behavior doesn’t happen.

> Compilers can assume undefined behavior doesn't happen.

As far as I understand it we cannot check for integer overflow by constructing code which overflows it, as it is undefined behavior, and the compilers can assume that we will not do it.

* Loop unrolling - save work by combining several iterations of a loop into a single iteration, reducing the number of iterations and the number of times the condition has to be recomputed.

There is partial and full loop unrolling. 

"Polluting the instruction cache"

* Loop fusion - (also called "jamming") combine multiple loops into a single loop body, thereby saving the overhead of loop control.

It can allow better cache locality if the two loops share some common elements.

* Eliminating wasted iterations avoid executing loop iterations over empty loop bodies.

### Functions

* Inlining - avoid the overhead of calling a function by replacing a call with the body of a function itself. 

Compiler is pretty good at doing it, even if we don't declare function as a static inline it can still figure it out.

* Tail recursion elimination - replace recursive call that occurs as a last step of a function with a branch, saving function call overhead.

* Coarsening recursion - increase the size of the base case and handle it with more efficient code that avoids function-call overhead.

Avoid premature optimization, first get correct working code, then optimize whil preserving the correctness by regression testing.

Reducing the work of a program does not necessarily decrease it's running time, but it's a good heuristic.

Compiler automates many low-level optimizations.

To verify if optimization is done one should look at the assembly code outputted.

## 3. Bit hacks

Bit manipulation idioms are really interesting, bit shifting by k (1 << k) and then changing a bit to zero or one can be done in a very elegant way.

Set kth bit in a word x to 1:

` y = x | (1 << k)`

Clear the kth bit in a word x:

`y = x & ~(1 << k)`

Flip the kth bit:

`y = x ^ (1 << k)`

Extract a bit field from word x:

`(x & mask) >> shift` -> Very useful for encoding and decoding data

Set a bit field in a word x to a value y:

`x = (x & ~mask) | (y << shift)`

Bit variable swap

``` C
x = x ^ y
y = x ^ y
x = x ^ y
```

Still not clear on how exactly this works.

This code is poort at exploiting instruction-level parallelism, ends up with worse performance than a temporary variable.

Find minimum of two integers without using a branch:

```
r = y ^ ((x ^ y) & -(x < y))
```

`(x < y)` evaluates to 0 or 1

`y ^ ((x ^ y)` evaluates to x.

`__restrict` tells the compiler that this is the only pointer that can point to specific piece of data.

`cmove` branchless instruction for comparisons

Modular addition

`(x + y) mod n` 

Division is expensive, unless by a power of 2.

```
z = x + y;
r = z - (n & -(z >= n))
```

Branchless solution which looks like the least expensive way to compute the modular addition.

Round up to a power of 2

Subtract 1 from the value, bitshift until all the bits to the right of leftmost bit are poppulated by 1s and add 1 after to get 2^[lg n]
We subtract to make sure that we round up to the same value if we have a value which is a power of 2 already.

Computer the mask of the least-significant 1 in word x.

`r = x & (-x)`  

Some card trick all of a sudden in the video?

deBruijn sequence - cyclic sequence of length 2^k such that each of 2^k 0-1 strings of length occurs exactly once as a substring of s.
It can be used to find the log base 2 of a power of 2.

There is a proof that for any length there is a deBruijn sequence.

This trick to computer the log base 2 of a power of 2 is nowadays turned into a processor instruction, so there is no need to implement it in software.

Approximate cost for different levels of memory hierarchy per 64-byte cache line:
* register: 1 cycle
* L1-cache: 4 cycles
* L2-cache: 10 cycles
* L3-cache: 50 cycles
* DRAM: 150 cycles

Calculating pop count (number of bits set to 1)

Parallel divide-and-conquer - completely didn't get it, would need to rewatch it.
```
M5 = 
```

Most modern machines provide popcount instructions which operate much faster than any custom solutions.
Compier intrinsics might allow one to use them.

## Assembly Language & Computer Architecture

Loved the joke about Intel.

There is a whole system around the assembly instructions names, there is an opcode but there are also extensions which describe data types and even additional operations.

`movzbl %al %edx` `mov` - moves (copies) the data, `z` will extend it with zeroes (because we are moving two different bit-size integers), `b` says that the destination will be a single byte and `l` says that the source is a double word (32 bits/4 bytes)

`RFLAGS` register - specific bits will set specific status flags 

Base indexed scale displacement.

`movq 172(%rdi, %rdx, 8), %rax` - frequently used when accessing stack memory.

`xor %rax, %rax` - zeros the register, very short instruction.

`test %rcx, %rcx` - will set zero flag if %rcx is zero, it's an idiom used for checking if value is equal to zero.

x86-64 ISA inclues several no-op instructions, including `nop`, `nop A` and `data16`.
Why would compiler generate idioms with noops?
Mainy to optimize instruction memory (e.g. code size, alignement)

SSE instructions use two-letter suffixes to encode the data type.
First letter is either `s`ingle (scalar) or `p`acked, second letter is either `s`ingle or `d`ouble precision.
* ss - one single-precision floating point value (float)
* sd - one double-precision floating point value (double)
* ps - vector of single-precision floating point values
* pd - vector of double-precision floating point values

Modern processors often incorporate vector hardware to process data in a single-instruction stream, multiple-data stream (SIMD) fashion.
Vector units - processor issues instructions to all vector units.

`v` in front of an opcode distinguishes AVX/AVX2 instructions.

The vector/scalar instructions are in:
* SSE
* AVX
* AVX2 - 256 regsiters
* AVX3 - 512 registers

A simple 5-Stage processor
* Instruction fetch (IF) - read the instruction from memory
* Instruction Decode (ID) - determine which units to use to execute the instruction and extract the register arguments
* Execute (EX) - perform ALU operations
* Memory (MA) - read/write data memory
* Write back (WB) - store results into regsiters

Intel Haswell has actually 14-19 pipeline stages, it's vastly more complicated.
There are different paths through it, so the number might be different.

Historically computer architects have aimed to improve processor performance by two means:
* Exploit parallelism by executing multiple instructions simultaneously.
* Explot licality to minimize data movement (i.e. caching)

Instruction level parallelism - pipelining the 5 stages of processor to happen simultaneously. In practice various issues can prevent an instruction from executing during its designated cycle, cusing the processor pipeline to stall.

Stalls happen because of `hazards`
* Structural hazard - two instructions attempt to use the same functional unit at the same time.
* Data hazard - an instruction depends on a result of a prior instruction in the pipeline.
    * True dependence (RAW - Read After Write) - instruction i write to a location that instruction j reads
    * Anti-dependence (WAR - Write After Read) - instruction i reads a location that instruction j writes.
    * Output-dependence (WAW - figure it out) - both instructions i and j want to write to the same location
        Why would two things want to move data to the same location? To set some flags or for value aliasing (what is it?)
* Control hazard - fetching and decoding the next instruction to execute is delayed by a decision about control flow

Some arithmetic operations are more complex and have longer latencies.
Most integer arithmetic, logic, shift - 1 cycle
Integer multiply - 3 cycles
Integer division - variable
Floating-point add - 3 cycles
Floating-point multiply - 5 cycles
Floating-point divide-  variable
Floating-point FMA (Multiply-Add) - 5 cycles - useful operation for matrix operations

Hardware has complex pipelines with separate functional units for different operations, such as floating-point arithmetic.
There are separate registers (like XMM) for  floating point registers and other specific values.

Idea to keep the pipeline busy: fetch multiple instructions per cycle.
Super scalar processor - executes multiple instructions at a time.
Haswell breaks up x86-64 instructions into simpler operations called micro-ops.
The fetch and decode stages can emit 4 micro-ops per cycle to the rest of the pipeline.
They also implement optimizations on micro-op processing, including special cases for common patters like xor %rax, %rax. (in the example it doesn't even use functional unit to zero rax)

Out-of-order-execution

Bypassing - stategy that allows an instruction to read its arguments before they've been stored in a general purpose register.

Eliminating data and name dependencies

Scoreboarding - complex mechanism used to change the register names to eliminate WAR dependencies.

### Branch prediction

Branch misprediction effect is just like stalling, on Haswell misprediction costs about 15-20 cycles.

## C to Assembly

Explanation of LLVM IR, it has smaller set of instructions than x86

IR body function is partitioned into basic blocks, sequences of instructions where control only enters through the first instruction and exits only from the last.

Control-flow instructions induce control flow edges between the basic blocks of a function, creating a control-flow graph (CFG).

`phi` instruction in LLVM IR exists to solve a problem in the representation of loops. It's not a "real" processor instruction, it's just an LLVM construct to solve a problem of static single assignment where each register name is written on at most one line of IR.

LLVM Attributes - tags added to give some additional info to compiler about given operations, it enables more optimization.

Compared to C, all operations in IR are explicit. No implicit conversion, no type casts between two values.

Compiler has to perform three tasks to translate LLVM IR into x86-64 assembly:
* Select assembly instructions to implement LLVM IR instructions
* Allocate x86-64 general-purpose registers to hold values
* Coordinate function calls

### Linux x86-64 calling convention

Assembler directives - refer to and operate on sections of assembly.
Segment directives - organize the contents of an assembly file into segments.
```
.text               identifies text segment
.bss                identifies bss segment
.data               identifies data segment
```

Storage directives - store content into the current segment.

```
x:  .space 20 0     Allocates 20 bytes at location `x`
y:  .long 172       Stores the constant 172L at location `y`
z:  .asciz "6.172"  Store the string "6.172\0" at location `z` 
    .align 8        Align the next content to an 8-byte boundary
```

Scope and linkage directives - control linking

```
.globl fib          Makes "fib" visible to other project files
```

What exactly ends up in a stack segment?
- Local variables, function arguments, return address, "intermediate results" a function might want to stash some register state values to use them later when it doesn't have enough registers available.

How do functions in different object files coordinate their use of the stack and of register state?
Functions abide by a calling convention.

Linux x86-64 calling convention organizes stack into frames, where each function instantiation gets a single frame of its own.

`%rbp` points to the top of the current stack frame, `%rsp` points to the bottom.

I was wondering why is there a set, small number of registers in the CPU, below is great answer from SO.

    > Why does a processor have 32 registers?

    First, not all processor architectures stopped at 32 registers. Almost all the RISC architectures that have 32 registers exposed in the instruction set actually have 32 integer registers and 32 more floating point registers (so 64). (Floating point "add" uses different registers than integer "add".) The SPARC architecture has register windows. On the SPARC you can only access 32 integer registers at a time, but the registers act like a stack and you can push and pop new registers 16 at a time. The Itanium architecture from HP/Intel had 128 integer and 128 floating point registers exposed in the instruction set. Modern GPUs from NVidia, AMD, Intel, ARM and Imagination Technologies, all expose massive numbers of registers in their register files. (I know this to be true of the NVidia and Intel architectures, I am not very familiar with the AMD, ARM and Imagination instruction sets, but I think the register files are large there too.)

    Second, most modern microprocessors implement register renaming to eliminate unnecessary serialization caused by needing to reuse resources, so the underlying physical register files can be larger (96, 128 or 192 registers on some machines.) This (and dynamic scheduling) eliminates some of the need for the compiler to generate so many unique register names, while still providing a larger register file to the scheduler.

    There are two reasons why it might be difficult to further increase the number of registers exposed in the instruction set. First, you need to be able to specify the register identifiers in each instruction. 32 registers require a 5 bit register specifier, so 3-address instructions (common on RISC architectures) spend 15 of the 32 instruction bits just to specify the registers. If you increased that to 6 or 7 bits, then you would have less space to specify opcodes and constants. GPUs and Itanium have much larger instructions. Larger instructions come at a cost: you need to use more instruction memory, so your instruction cache behavior is less ideal.

    The second reason is access time. The larger you make a memory the slower it is to access data from it. (Just in terms of basic physics: the data is stored in 2-dimensional space, so if you are storing n
    bits, the average distance to a specific bit is O(n−−√).) A register file is just a small multi-ported memory, and one of the constraints on making it larger is that eventually you would need to start clocking your machine slower to accommodate the larger register file. Usually in terms of total performance this is a lose. 

    Source: https://cs.stackexchange.com/a/22591

I will need to rewatch the last segment about calling convention, it makes sense but I don't fully understand it.

Loved the presentation showing the translation of LLVM IR to assembly.