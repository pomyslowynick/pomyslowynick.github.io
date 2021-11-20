---
layout: post
title:  "Computer Networking - Top Down Approach"
date:   2021-05-19 20:20:45 +0100
categories: networking learning ossu 
---
This is an introductory text to computer networking which is something I really need to ramp up on, my knowledge of it is incredibly limited. On this page I will publish my notes about each one of the chapters.
## First chapter
Introduction to what internet is as well quick synopsis of the chapters to come. 
*Hosts* or *End systems* - devices connected to the internet.
> End systems are connected together by a network of *communication links* and *packet switches*.

On the physical level networks are connected through different types of cables (fiber, copper, radio, coaxial) that can transmit data at different rates. Bits/second rate is called transmission rate.
The data sent from one system to another along with it's headers is called a *packet*.

Packet switches receive the packet and forward it to one of it's outgoing communication links. The two most prominent types of switches are *routers* and *link-layer switches*.
The sequence of communication links that a packet takes is called a *route* or a *path*.

Memorizing network elements through the truck logistics can be a good mnemonic technique.

End systems acces the Internet through ISPs which can be: residential, cellular, univeristy, corporate. Those are lower level ISPs which then are connected to national and global level ISPs.

Internet Engineering Task Force develops Internet standards and releases documents called RFCs (Request for Comment)

### Applications view
Internet can also be treated as an infrastructure that provides services to applications, allows them to communicate or exchange data.

*Socket interface* - specifies how a program running on one end system asks the Internet infrastructure to deliver data to a specific destination running on another end system.

> A protocol defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event.

> End systems are also referred to as hosts because they host (that is, run) appli-
cation programs such as a Web browser program, a Web server program, an e-mail client program, or an e-mail server program.

Hosts can be further split into *clients* and *servers*.

Edge router is the first step between the hosts at the edge and the Internet.

Two most prevalent types of broadband residential access - DSL (digital subscriber line) and cable. DSL makes use of existing telco infrastructure to convert data sent at certain frequencies to an analog signal. Cable uses existing television infrastructure to transfer the internet data.

> Telecommunications  companies  have  made  enormous  investments  in  so-called fourth-generation (4G) wireless, which provides real-world download speeds of up to 60  Mbps.  But  even  higher-speed  wide-area  access  technologies—a  fifth-generation (5G)  of  wide-area  wireless  networks—are  already  being  deployed.

> For each transmitter-receiver pair, the  bit  is  sent  by  propagating  electromagnetic  waves  or  optical  pulses  across a physical  medium

Electromagnetic waves or optical pulses are two ways information is sent in the Internet.

Most packet switches use *store and forward transmission*. It means that the packet switch must receive the entire ticket before it can begin to forward first bit of the packet.

When packets are rady to be transmitted through the output link they are stored in output buffer (also called output buffer). Packets waiting to be sent in the output queue incur *queueing delay*.

If the queue is full and a new packet arrives, then packet loss occurs.

In the Internet every end system has an address called IP address.

Internet has routing protocols which are used to automatically set the forwarding tables.
Those may determine the shortest path from router to a destination.

Two fundamental approaches to moving data in networks for links and switches: circuit switching and packet switching.

Circuit switching is still used by the telcosm although they are slowly moving to the packet switching too.

PoP - point-of-presence - group of one or more routers at the same location in the provider's network where customers ISPs can connect into ithe provider ISP. Simply a point where ISPs can connect.

IXP - internet exchange point - multiple ISPs can peer with each other at such a point.

Content Provider Networks - big companies like Google have their own data centers and communications set up, they can peer with local providers instead of paying tier-1 ISPs. In some regios they can't bypass tier-1 ISPs, but in general they exert much higher control over their network design and architecture.

Important delays tyoes in path from host to another host: 
* nodal processing delay
* queuing delay
* transmission delay
* propagation delay

Together, these delays accumulate to give a total *nodal delay*.

Processing delay is the time required to examine the packets header and determine where it needs to be directed. This is usually on order of microseconds or less in high speed routers.
After examining the header packet is then put into the queue that precedes the link to router B.

Queueing delay in practice can be on the order if microseconds to miliseconds.

Transmission delay - the packet can only be transmitted after all the packets that have arrived before have been trasnmitted. Typically on order of microseconds to miliseconds.

Propagation delay - how fast can a bit propagate through a physical medium.

Queueing delay is one of the most complicated and interesting metrics in packet processing, there have been numerous books and PhD thesis written about it.

Golden rule in networking in regards to queueing - "Design your system so that the traffic intensity if no greater than 1."
Typically the arrival process to a queue is random (I would think there have to be some statistical trends, like ones that have to do with daytime hours)

I really like the highway congestion example, it shows how the bigger gets the congestion the higher the overall delay is.

Performance of a node is often measured in the terms of the probability of packet loss.