---
layout: post
title:  "My progress in OSSU curriculum"
date:   2021-11-02 15:20:45 +0100
categories: self-development computer-science 
---

My notes on book Hands-On Network Programming with C

1. What are the key differences between IPv4 and IPv6?
The biggest difference is the expanded address space, IPv4 is 32bytes where IPv6 is 128 bytes long, allowing for more addresses.
IPv6 also adds new attributes like `scope` and `lifetime`.
2. Are the IP addresses given by the ipconfig and ifconfig commands the same IP addresses that a remote web server sees if you connect to it?
No, the addresses listed by ip/ifconfig are the local addresses which then get translated by NAT, probably in a router.s
3. What is the IPv4 loopback address?
127.0.0.1
4. What is the IPv6 loopback address?
`::1`
5. How are domain names (for example, example.com) resolved into IP addresses?
DNS (Domain Name System) servers are services which will translate the IP addresses into domain names. 
Host send a UDP message to a domain server to request AAAA-type record for the domain we're trying to resolve. If it doesn't exist it will try the same for an A record, which would give an IPv4 address.
6. How can you find your public IP address?
By using external websites which can return my external IP.
7. How does an operating system know which application is responsible for an incoming packet?
Those connections will be established on specific ports, which then in turn have sockets associated with them. A socket is a tuple of five values, local IP address, local port, remote IP address, remote port and a protocol (either UDP or TCP).
What differentiates applications on a single machine which want to use the same remote port is the local port, also called the ephemeral port.

### Chapter 2

> Sockets come in two basic types—connection-oriented and connectionless.

STREAM socket is TCP oriented where DGRAM is for the UDP.

`getaddrinfo` is a function with many purposes, in the book we use it to generate an address suitable for bind operation. It can also do the DNS lookups if the first argument is a domain name.

> It is also possible for the listening IPv6 socket to accept IPv4 connections with a dual-stack
socket. Not all operating systems support dual-stack sockets. With Linux in particular,
support varies between distros. 

`inetd` - Linux service which can be used to turn a console application into a networked one.


1. What is a socket?
Socket is an abstract "object", structure?, which consists at the very five values, a local IP address, local port, remote port, remote address and a protocol (UDP or TCP).
It acts as a part of networking interface for an operating system, by accepting, listening to, receiving and sending data on specific ports.
> A socket is an abstraction that represents one endpoint of a communication link
between systems.

2. What is a connectionless protocol? What is a connection-oriented protocol?
UDP is connectionless, TCP is connection-oriented. First sends the data withotu first establishing the connection between the host, it gets the address, sends the packets and forgets about that them. TCP is a type of protocol which first establishes the connection with the host through a series of handshakes, makes sure it has delivered all the packets and resends them in case of a failure.
> A connection-oriented protocol sends data packets in the context of a larger
stream of data. A connectionless protocol sends each packet of data
independently of any before or after it.

3. Is UDP a connectionless or connection-oriented protocol?
UDP is connectionless.

4. Is TCP a connectionless or connection-oriented protocol?
TCP is connection-oriented.

5. What types of applications generally benefit from using the UDP protocol?
Streaming, on call audio transfers, anything real time which shouldn't be resent. On the other hand small payloads like resolving DNS can be sent through UDP as it will be obvious is packet has not been sent, and we know that the whole payload can be sent in just one packet. 

6. What types of applications generally benefit from using the TCP protocol?
Anything that requires data integrity when connected, sending website data, video, FTP transfers.

7. Does TCP guarantee that data will be transmitted successfully?
It does guarantee that all of the packets will be sent and that it will try to resend the missing ones.

8. What are some of the main differences between Berkeley sockets and Winsock sockets?
Winsock requires the whole WSA module startup and clean up, it doesn't treat the sockets like files (which is what Unix systems do with Berkeley sockets). Winsock implementations are Windows specific, so they are more or less the same between Windows versions. Berkeley sockets have different implementations which vary between Unix systems.

> The header files are different. Sockets themselves are represented as signed
versus unsigned ints. When socket() or accept() calls fail, the return values
are different. Berkeley sockets are also standard file descriptions. This isn't always
true with Winsock. Error codes are different and retrieved in a different way.
There are additional differences, but these are the main ones that affect our
programs.

9. What does the bind() function do?
This function binds a specific socket to a port.

> The bind() function associates a socket with a particular local network address
and port number. Its usage is almost always required for the server, and it's
usually not required for the client.

10. What does the accept() function do? 
The accept function will accept incoming "client" connections and create a new socket for each new connection so that the server can listen to new connections.

> The accept() function will block until a new TCP client has connected. It then
returns the socket for this new connection.

11. In a TCP connection, does the client or the server send application data first?
It can work both ways, either the client or server can send the data first depending on the arrangement. This is contrary to UDP.

> Either the client or the server can send data first. They can even send data
simultaneously. In practice, many client-server protocols (such as HTTP) work by
having the client send a request first and then having the server send a response.

