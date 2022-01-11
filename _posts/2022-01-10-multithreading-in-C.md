---
layout: post
title:  "Multithreading in C"
date:   2022-01-10 22:39:12 +0100
categories: c programming computer-science 
---


[Multithreading in C](https://www.youtube.com/watch?v=fXWgsC31RxQ)

Another video from "Together C and C++" conference, I like how relevant they stay to my everyday work.
It uses the UNIX pthread interface.

Amdahl's law puts an upper bound on how much speed up can be achieved with additional threads. Depending on how much work can be parallelized we might achieve somewhere between 20x and 2x, if there will be benefit at all. 20x is the upper bound of performance improvement.