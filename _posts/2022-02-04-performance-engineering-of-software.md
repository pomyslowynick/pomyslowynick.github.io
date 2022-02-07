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

Packing - fit more than one data value in a machine word, related to the idea of encoding when we convert data values into a representation that requires fewer bits. 

``` C
typedef struct {
    int year: 13;
    int month: 4;
    int day: 5;
} date_t;
```

You can define the amount of bits each field within the struct should take after the colon, like in the example above when it adds up to 22 bits which have a structure that can be easily used to retrieve the data.

Sometimes unpacking and decoding can result in better performance, depending on the access patterns.