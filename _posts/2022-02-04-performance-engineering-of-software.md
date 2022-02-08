---
layout: post
title:  "MIT 6.172 Performance Engineering of Software Systems, Fall 2018"
date:   2022-02-04 23:16:12 +0100
categories: course notes performance hardware benchmarking profiling 
---

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