---
title: "Linux Series: Sockets"
date: 2025-05-28
---

Sockets are a method of IPC that allow data to be exchanged between applications, either on the same
host (computer) or on different hosts connected by a network

## Overview

Applications communicate using sockets as follows:

- Each application creates a socket. A socket is the "apparatus" that allows communication, and both
  applications require one
- The server binds its socket to a well-known address (name) so that clients can locate it

```c
fd = socket(domain, type, protocol)
```

### Communication domains

Sockets exists in a _communication domain_, which determines:

- the method of identifying a socket
- the range of communication

Modern operating systems support at least the following domains:

- The UNIX (AF_UNIX) domain allows communication between applications on the same host
- The IPv4 (AF_INET) domain allows communication between applications running on hosts connected via
  an Internet Protocol version 4 (IPv4) network
- The IPv6 (AF_INET6) domain allows communication between applications running on hosts connected via
  an Internet Protocol version 6 (IPv6) network

| Domain   | Communication performed | Communication between applications     | Address format                            | Address structure |
| -------- | ----------------------- | -------------------------------------- | ----------------------------------------- | ----------------- |
| AF_UNIX  | within kernel           | on same host                           | pathname                                  | _sockaddr\_un_    |
| AF_INET  | via IPv4                | on hosts connected via an IPv4 network | 32-bit IPv4 address + 16-bit port number  | _sockaddr\_in_    |
| AF_INET6 | via IPv6                | on hosts connected via an IPv6 network | 128-bit IPv6 address + 16-bit port number | _sockaddr\_in6_   |

### Socket types

Every sockets implementation provides at least two types of sockets: stream and datagram. These docket
types are supported in both the UNIX and the Internet domains

| Property                      | Stream | Datagram |
| ----------------------------- | ------ | -------- |
| Reliable delivery?            | Y      | N        |
| Message boundaries preserved? | N      | Y        |
| Connection-oriented?          | Y      | N        |

_Stream sockets_ (SOCK_STREAM) provide a reliable, bidirectional, byte-stream communication channel.
Stream sockets operate in connected pairs. For this reason, stream sockets are described as
_connection\-oriented_.

_Datagram sockets_ (SOCK_DGRAM) allow data to be exchanged in the form of messages called _datagrams_.
With datagram sockets, message boundaries are preserved, but data transmission is not reliable. Messages
may arrive out of order, be duplicated, or not arrive at all. Datagram sockets are an example of
_connectionless_ socket.

## Stream Sockets

Stream sockets are distinguished as being either active or passive:

- A socket that has been created using _socket()_ is _active_. An active socket can be used in a _connect()_
  call to establish a connection to a passive socket. This is referred to as performing an _active open_
- A _passive_ socket (also called a _listening_ socket) is one that has been marked to allow incoming
  connections by calling _listen()_. Accepting an incoming connection is referred to as performing a 
  _passive open_

![Stream Sockets](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/stream_sockets.drawio.png)

### I/O on Stream Sockets

A pair of connected stream sockets provides a bidirectional communication channel between the two endpoints

The semantics of I/O on connected stream sockets are similar to those for pipes:

- To perform I/O, we use the _read()_ and _write()_ system calls (or the socket-specific _send()_ and
  _recv()_). Since sockets are bidirectional, both calls may be used on each end of the connection
- A socket may be closed using the _close()_ system call or as a consequence of the application
  terminating. Afterward, when the peer application attempts to read from the other end of the connection,
  it receives end-of-file (once all buffered data has been read). If the peer application attemps to
  write to its socket, it receives a _SIGPIPE_ signal, and the system call fails with the error _EPIPE_

![UNIX Domain Stream Sockets](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/unix_domain_stream_sockets.drawio.png)

## Datagram Sockets

![Datagram Sockets](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/datagram_sockets.drawio.png)

### Using _connect()_ with Datagram Sockets

Even though datagram sockets are connectionless, the _connect()_ system call serves a purpose when
applied to datagram sockets. Calling _connect()_ on a datagram socket causes the kernel to record a
particular address as this socket's peer. The term _connected datagram socket_ is applied to such a
socket.

After a datagram socket has been connected:

- Datagrams can be sent through the socket using _write()_ (or _send()_) and are automatically sent
  to the same peer socket
- Only datagrams sent by the peer socket may be read on the socket

Advantages of setting peer for a datagram is simpler I/O system calls when transmitting data on the socket
