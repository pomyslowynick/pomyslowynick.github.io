---
layout: post
title:  "Adventures with the eBPF Data Plane"
date:   2022-02-13 12:17:12 +0100
categories: operating-systems notes eBPF calico kubernetes
---

### Episode 1
My notes on the video series 
https://www.youtube.com/watch?v=aj9ZrnHYOCc

* There are multiple data planes
* Both Control Plane and Data Plane exist in a same physical device
    * Control Plane has different requirements and workloads,
    * Data Plane sends the actual content across the network 

* In an ideal scenario control plane is decoupled from the data plane
* If the things are built the right way the two can be separated
Data Plane has to be extremely reliable, it is achieved by making it as small as possible, testing it thouroughly, using tested software like Linux kernel to parse the packets.
COntrol Plane doesn't need to be as reliabel, it can reset and have it's config upgraded while the packet are flowing. 

In the data plane changing the config on the fly is very challenging, as the packets are flying.

COntrol Plane needs to run on more traditional CPUs, Data Plane since it's more predictable in its size and smaller can be run on ASIC implementation or any hardware acceleration a device offers.

COntrol Plane runs in user space, the data plane runs in the kernel.

ASIC running data plane can have massive performance boost by restricting the instruction set. General Purpose CPUs are slower, but needed for control plane.

COntrol and data plane software are easy to audit as not much changes in it's protocols.

* VPP can be the implementation of the Data Plane

The most interesting data in an IPv4 packet are usually in the first 20 bytes.
Header checksum has to be recalculated each time header changes.

Above my head: There is considerable amount of complexity in determining the path with variable length subnets when looking up the route for a packet. Will need to ponder on this one a bit.

"Longest prefix match" - is what you want to do to determine the route. 
It is generally not an easy thing to do for a general purpose machine, you need to build a tree of addresses that fan out of this tree.

I understand it better now, at least I think so. I thought that subnet routing was basically comparing two values, but it looks like it is actually a tree traversal/search.

ASIC can dedicate hardware to perform massive parallel lookup of the variable length addresses to quickly get an answer.

Price tags of modern routers and switches are high.

[MPLS](https://en.wikipedia.org/wiki/Multiprotocol_Label_Switching) Header - fixed length, initial label of 20 bits, EXP, stack label and TTL

It's just table lookup for MPLS, there is no checkum no heavy workload to modify it.
In data plane there is much less complexity, it's cheap to forward on IPv4 devices.

MPLS allows prerouting, so that hardware knows which routing table to use even in case of hardware failure.

eBPF is a sort of new data plane. eBPF gives better performance, source IP preservation, takes out the need for kube-proxy. 

Disadvantage is in the interoperability, ICAN?

IPtables based dataplane uses existing kernel hooks to filter traffic, for NATing traffic, routing, SNATing (NAT as packets leave the cluster to masquarede). eBPF opens up possibilities in Kernel that in the past would be very hard to introduce, new level of experimentation. 

Netfiler is the kernels underlying architecture to filter and manipulate the packets. 
There has been multiple generations of APIs on top of it, the current one is Nftables, previous one was IPtables. It basically comes as a chain of hooks with which you can do specific things with a packet.

Flow based tracking in Calico. 

Kube-proxy hooks into pre-routing NAT hook in the network layer where it inputs it's rules.
Proxy basically influences the routing decision to be forwarded to a veth interface.

Mangle forward chain?  

Forward filter chain is where Calico policies come in, this is where the IP address has been NATed.

A lot of eBPF performance comes from the fact that there is no Netfilter rules to be examined against.

The downside of netfilter is that a packet goes through stages it mind not need.


BPF works on a principle of attaching hooks to the kernel.

BPF hook runs in the network driver, before anything happens in the kernel the hooks (XDP hook) will already intercept the packet and influence it's route, destination, filter etc.

VPP uses XDP program that captures the packet and sends it to VPP where it processes it.

Different buckets of eBPF hooks:
* Functional hooks (like XDP)
    * Redirecting the packets to user space (a little sandbox of options)
* K-probes (there are milions of them in the kernel)
    * Read only, used for observability, can be attached to collect stats about packets.

No buffers are allocated for packets dropped from the XDP hooks, making it great DDOS mitigation.

XDP hook comes early in the packet flow, but it doesn't provide as much capabilities as the ingres(qdisc) hook, which is used in Calico eBPF. Tunneling can be implemented there, routing can be done there too. The eBPF hook will also skip the kernel processing if it recogises a packet, it will forward it right to the veth associated with it in the past. It can then skip the steps in the middle on egress.

Routing is quite complicated.

> Egress in the world of networking implies traffic that exits an entity or a network boundary, while Ingress is traffic that enters the boundary of a network.