---
layout: post
title:  "Notes from: The Art Of Writing Efficient Programs"
date:   2022-01-06 18:20:45 +0100
categories: cpp systems performance
---

My notes on an incredibly interesting book, working my way through it.

---

Chapter 1 - Introduction

Reasons why performance is increasingly more important are outlined and there is an example of an area in which the computing performance had to be kept at it's bleeding edge: EDA or Electronic Design Automation. Verifying that processor are working according to their specification requires robust test performed by an another processor, most likely the most efficient model of last years one.

Introduced performance metrics:
* Throughput - how many operations can be executed in a given timeframe.
* Turnaround time - how long it takes to complete an operation.
* Latency - delay between data being ready and when processing is completed.
* Latency tail - percentile of the delay, example if *t* is the 95th percentile latency than 95% of all computations take less time than *t*.

Importance of gathering metrics: *First rule of performance is to never guess about performance*

What makes the program efficient?
* Choosing the right algorithm
* Using CPU resources effectively
* Using memory effectively
* Avoiding unnecessary computations
* Using concurrency and multi-threading effectively
* Using the programming language effectively, avoiding inefficiencies
* Measing performance and interpreting results

Questions:
1. The advances in processing power are no longer growing at the rate they used to and as such performance is a bigger concern. Program performance is not only about the amount of computations that can be done in a given timeframe, depending on a metric it could mean a different thing. Because of that it's still important to design performant code in some areas of computing.
2. One needs to understand the programming langauge they are using to avoid it's inefficiencies and write performant code. The knowledge of hardware allows one to use the underlying components to their full potential by writing code which runs efficiently.
3. Performance can be any given metric, for example how well a program performs in throughput. Efficiency is how well a program utilizes underlying resources like a processor or GPU.
4. Because without performance metrics we don't really know what we should optimize for. And different applications have different end goals.
5. By comparing the specified metrics before and after optimizing for them and veryfing that program is working as expected.
