---
layout: post
title:  "Graviton2 and Graviton3 - Notes on Article"
date:   2022-03-24 20:58:10 +0100
categories: notes datacenters wsc amazon murat
---

My (short) notes on Murat Demirbas transcript of [Graviton2 and Graviton3 talks](https://muratbuffalo.blogspot.com/2021/12/graviton2-and-graviton3.html)

* Dennard scaling - observation that processors got more power efficient with time, allowing for higher frequencies at the same power input. It ties in again into the graph of individual technologies offering diminishing returns, I really like looking at tech through this Keller analogy.

Graviton3 has increased the width of the core, the number of instructions that each core can work on concurrently went from 5 to 8. This obviously will only benefit the workloads if the pipelining is done efficiently, i.e. there has to be little data dependencies in the computation graph. I would think that the numbers other companies have used so far have been a middle ground between different types of workloads.

In general there is a lot of mentions about CSP specific workloads and measuring the performance against them more so than exploring metrics like frequency tuning or number of cores. It's a natural sales pitch but also the key point of Gravitons, Amazon knows it's hardware requirements and can heavily optimize their hardware designs for them.

Is the future of computing hardware companies to become foundries with highly customizable parameters? From this perspective tt looks like software is the driver for the design and hardware is trying to fit into it's requirements.

It's not that suprising when I think about the FANG companies, or whatever the current acronym is. They primarily software oriented, and so are the biggest startups. Hardware is the platform.

