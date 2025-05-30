---
title: "Linux Series: Internet Domain Sockets"
date: 2025-05-30
---

## Internet Domain Sockets

Internet domain stream sockets are implemented on top of TCP. They provide reliable, directional, byte-stream
communication channel. Internet domain datagram sockets are implemented on top of UDP, but not like the
datagram in UNIX domain, Internet domain datagram sockets are not reliable. Furthermore, when the receiving
socket is full, sending on UNIX domain datagram socket will block, but it is silently dropped in Internet
domain datagram sockets

## Network Byte Order

IP addresses and port numbers are integer values. Different hardware architectures store the bytes of 
a multibyte integer in different orders. The byte ordering used on a particular machine is called the
_host byte order_. Since port numbers and IP addresses must be transmitted between, and understood by,
all hosts on a network, a standard ordering must be used. This ordering is called _network byte order_,
and happens to be big endian. We need to convert between _host byte order_ and _network byte order_
before using

```c
#include <arpa/inet.h>

// return host_uint16 converted to network byte order
uint16_t htons(uint16_t host_uint16);

// return host_uint32 converted to network byte order
uint32_t htonl(uint32_t host_uint32);

// return net_uint16 converted to host byte order
uint16_t ntohs(uint16_t net_uint16);

// return net_uint32 converted to host byte order
uint32_t ntohl(uint32_t net_uint32);
```

## Internet Socket Addresses

Two types of Internet domain socket addresses: IPv4 and IPv6

```c
// IPv4 4-byte address
struct in_addr {
    // unsigned 32-bit integer
    in_addr_t s_addr;
};

// IPv4 socket address
struct sockaddr_in {
    sa_family_t sin_family; // address family (AF_INET)
    in_port_t sin_port; // port number
    struct in_addr sin_addr; // IPv4 address
    unsigned char __pad[X]; // pad to size of 'sockaddr' structure (16 bytes)
};
```

```c
// IPv6 address structure
struct in6_addr {
    // 16 bytes = 128-bit address
    uint8_t s6_addr[16];
};

// IPv6 socket address
struct sockaddr_in6 {
    sa_family_t sin6_family; // address family (AF_INET6)
    in_port_t sin6_port; // port number
    uint32_t sin6_flowinfo; // IPv6 flow information
    struct in6_addr sin6_addr; // IPv6 address
    uint32_t sin6_scope_id; // scope ID
};
```

## Host and Service Conversion Functions

A _hostname_ is the symbolic identifier for a system that is connected to a network (possibly with
multiple IP addresses). A _service name_ is the symbolic representation of a port number.

Two methods are available for representing host addresses and ports:

- A host address can be represented as a binary value, as a symbolic hostname, or in presentation format
  (dotted-decimal for IPv4 or hex-string for IPv6)
- A port can be represented as a binary value or as a symbolic service name

_getaddrinfo()_ function is the modern successor to both _gethostbyname()_ and _getservbyname()_. Given
a hostname and a service name, _getaddrinfo()_ returns a set of structures containing the corresponding
binary IP address(es) and port number. Unlike _gethostbyname()_, _getaddrinfo()_ transparently handles
both IPv4 and IPv6 addresses. Thus, we can use it to write programs that don't contain dependencies on
the IP version being employed

_inet\_pton()_ and _inet\_ntop()_ functions allow conversion of both IPv4 and IPv6 addresses between
binary form and dotted-decimal or hex-string notation

```c
#include <arpa/inet.h>

// return 1 on successful conversion, 0 if src_str is not in presentation format, or -1 on error
int inet_pton(int domain, const char *src_str, void *addrptr);

// return pointer to dst_str on success, or NULL on error
const char *inet_ntop(int domain, const void *addrptr, char *dst_str, size_t len);
```

The _p_ in the names of these functions stands for "presentation", and the _n_ stands for "network".
The presentation form is a human-readable string, such as the following:

- 204.152.189.116 (IPv4 dotted-decimal address)
- ::1 (an IPv6 colon-separated hexadecimal address)
- ::FFFF:204.152.189.116 (an IPv4-mapped IPv6 address)

## The /etc/services File

Well-known port numbers are centrally registered by IANA. Each of these ports has a corresponding
_service name_. Because service numbers are centrally managed and are less volatile than IP addresses,
an equivalent of the DNS server is usually not necessary. Instead, the port numbers and service names
are recorded in the file _/etc/services_. The _getaddrinfo()_ and _getnameinfo()_ functions use the
information in this file to convert service names to port numbers and vice versa

## UNIX Versus Internet Domain Sockets

There are some reasons why we may choose to use UNIX domain sockets:

- On some implementations, UNIX domain sockets are faster than Internet domain sockets
- We can use directory (and, on Linux, file) permissions to control access to UNIX domain sockets, so
  that only applications with a specified user or group ID can connect to a listening stream socket
  or send a datagram to a datagram socket. This provides a simple method of authenticating clients.
  With Internet domain sockets, we need to do rather more work if we wish to authenticate clients
- Using UNIX domain sockets, we can pass open file descriptors and sender credentials

## Server Design

Different designs including:

- _Iterative:_ the server handles one client at a time, processing that client's request(s) completely,
  before proceeding to the next client. _Iterative servers_ are usually suitable only when client
  requests can be handled quickly
- _Concurrent:_ the server is designed to handle multiple clients simultaneously. Concurrent servers
  are suitable when a significant amount of processing time is required to handle each request, or
  where the client and server engage in an extended conversation, passing messages back and forth

Besides, there are also other concurrent server designs:

- _Preforked and prethreaded servers:_ Instead of creating a new child process (or thread) for each
  client, the server precreates a fixed number of child processes (or threads) immediately on startup
  (i.e., before any client requests are even received). These children constitute a so-called _server pool_.
  Each child in the server pool handles one client at a time, but instead of terminating after handling
  the client, the child fetches the next client to be serviced and services it, and so on. This technique requires some careful management within the server application. The server pool should
  be large enough to ensure adequate response to client requests. This means that the server parent must
  monitor the number of unoccupied children, and, in times of peak load, increase the size of the pool
  so that there are always enough child processes available to immediately serve new clients. If the load
  decreases, then the size of the server pool should be reduced, since having excess processes on the
  system can degrade overall system performance
- _Handling multiple clients from a single process:_ To do this, we must employ one of the I/O models
  (I/O multiplexing, signal-drive I/O or _epoll_) that allow a single process to simultaneously monitor
  multiple file descriptors for I/O events. In a single-server design, the server process must take on
  some of the scheduling tasks that are normally handled by the kernel. In a solution that involves one
  server process per client, we can rely on the kernel to ensure that each server process (and thus client),
  gets a fair share of access to the resources of the server host. But when we use a single server process
  to handle multiple clients, the server must do some work to ensure that one or a few clients don't
  monopolize access to the server while other clients are starved
- _Using server farms:_ Handling high client loads involve the use of multiple server systems - a
  _server farm_. One simple approach is to use DNS round-robin load sharing. A more flexible, but also
  more complex, solution is _server load balancing_, in which a single load-balancing server routes
  incoming client requests to one of the members of the server farm

## _inetd_ (Internet Superserver) Daemon

The contents of _/etc/services_ contains hundreds of different services listed. This implies that a
system could theoretically be running a large number server processes. However, most of these servers
would usually be doing nothing but waiting for infrequent connection requests or datagrams. All of
these server processes would nevertheless occupy slots in the kernel process table, and consume some
memory and swap space, thus placing a load on the system

The _inetd_ daemon is designed to eliminate the need to run large numbers of infrequently used servers.
Using _inetd_ provides two main benefits:

- Instead of running a separate daemon for each service, a single process - the _inetd_ daemon - monitors
  a specified set of socket ports and starts other servers as required. Thus, the number of processes
  running on the system is reduced
- The programming of the servers by _inetd_ is simplified, because _inetd_ performs several of the steps
  that are commonly required by all network servers on startup

### Operation of the _inetd_ Daemon

The _inetd_ daemon is normally started during system boot. After becoming a daemon process, _inetd_
performs the following steps:

- For each of the services specified in its configuration file, _/etc/inetd.conf_, _inetd_ creates a
  socket of the appropriate type (i.e., stream or datagram) and binds it to the specified port. Each
  TCP socket is additionally marked to permit incoming connections via a call to _listen()_
- Using the _select()_ system call, _inetd_ monitors all of the sockets created in the preceding step
  for datagrams or incoming connection requests
- The _select()_ call blocks until either a UDP socket has a datagram available to read or a connection
  request is received on a TCP socket. In the case of a TCP connection, _inetd_ performs an _accept()_
  for the connection before proceeding to the next step
- To start the server specified for this socket, _inetd()_ calls _fork()_ to create a new process that
  then does an _exec()_ to start the server program. Before performing the _exec()_, the child process
  performs the following steps:
  - Close all of the file descriptors inherited from its parent, except the one for the socket on which
    the UDP datagram is available or the TCP connection has been accepted
  - Duplicate the socket file descriptor on file descriptors 0, 1, and 2, and close the socket file
    descriptor itself (since it is no longer required). After this step, the execed server is able to
    communicate on the socket by using the three standard file descriptors
  - Optionally, set the user and group IDs for the execed server to values specified in _/etc/inetd.conf_
- If a connection was accepted on a TCP socket in step 3, _inetd_ closes the connected socket (since
  it is needed only in the execed server)
- The _inetd_ server returns to step 2

### The _/etc/inetd.conf_ file

```
telnet      stream      tcp     nowait      root        /usr/sbin/tcpd      in.telnetd
```

Each line of _/etc/inetd.conf_ consists of the following fields, delimited by white space:

- _Service name:_ This specifies the name of a service from the _/etc/services_ file. In conjunction
  with the _protocol_ field, this is used to look up _/etc/services_ to determine which port number
  _inetd_ should monitor for this service
- _Socket type:_ This specifies the type of socket used by this service
- _Protocol:_ This specifies the protocol to be used by this socket
- _Flags:_ This field contains either _wait_ or _nowait_. This field specifies whether or not the
  server execed by _inetd_ takes over management of the socket for this service. If the execed server
  manages the socket, then this field is specified as _wait_. This causes _inetd_ to remove this socket
  fromt he file descriptor set that it monitors using _select()_ until the execed server exits
- _Login name:_ username from _/etc/passwd_
- _Server program:_ Pathname of the server program to be execed
- _Server program arguments:_ This specifies one or more arguments, separated by white space, to be
  used as the argument list when execing the server program

## Partial Reads and Writes on Stream Sockets

A partial read may occur if there are fewer bytes available in the socket than were requested in the
_read()_ call. In this case, _read()_ simply returns the number of bytes available.

A partial write may occur if there is insufficient buffer space to transfer all of the requested bytes
and one of the following is true:

- A signal handler interrupted the _write()_ call after it transferred some of the requested bytes
- The socket was operating in nonblocking mode (O_NONBLOCK), and it was possible to transfer only
  some of the requested bytes
- An asynchronous error occured after only some of requested bytes had been transferred

## The _shutdown()_ System Call

Calling _close()_ on a socket closes both halves of the bidirectional communication channel. Sometimes,
it is useful to close one half of the connection, so that data can be transmitted in just one direction
through the socket. The _shutdown()_ system call provides this functionality

```c
#include <sys/socket.h>

int shutdown(int sockfd, int how);
```

| Option    | Description                                                                                                                 |
| --------- | --------------------------------------------------------------------------------------------------------------------------- |
| SHUT_RD   | Close the reading half of the connection. Subsequent reads will return end-of-file. Data can still be written to the socket |
| SHUT_WR   | Close the writing half of the connection. Once the peer application has read all outstanding data, it will see end-of-file. |
|           | Subsequent writes to the local socket yield the _SIGPIPE_ signal and an _EPIPE_ error                                       |
| SHUT_RDWR | Close both the read and the write halves of the connection. This is the same as performing a SHUT_RD followed by a SHUT_WR  |

Note that _shutdown()_ doesn't close the file descriptor, even if _how_ is specified as SHUT_RDWR. To
close the file descriptor, we must additionally call _close()_.

## The _sendfile()_ System Call

In order to transfer a file via a socket, one technique is to read the file and then write it to the
socket to transfer the data. However, this approach is inefficient since two system calls are needed
to use: one to copy the file contents from the kernel buffer cache into user space, and the other
to copy the user-space buffer back to kernel space in order to be transmitted via the socket.

The _sendfile()_ system call is designed to eliminate this inefficiency. When an application calls
_sendfile()_, the file contents are transferred directly to the socket, without passing through user
space. This is referred to as a _zero\-copy_ transfer

![sendfile()](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/sendfile.drawio.png)

```c
#include <sys/sendfile.h>

ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

The _sendfile()_ system call transfers bytes from the file referred to by the descriptor _in\_fd_ to
the file referred to by the descriptor _out\_fd_. The _out\_fd_ descriptor must refer to a socket. The
_in\_fd_ argument must refer to a file to which _mmap()_ can be applied; in practice, this usually means
a regular file

## TCP State Machine

![TCP State Machine](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/tcp_sm.drawio.png)

## Socket Options

Socket options affect various features of the operation of a socket

```c
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);

int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
```

### The _SO\_REUSEADDR_ Socket Option

The _SO\_REUSEADDR_ socket option serves a number of purposes, but one of the most common use cases is
to avoid the _EADDRINUSE_ (address already in use) error when a TCP server is restarted and tries to
bind a socket to a port that currently has an associated TCP. There are two scenarios in which this
usually occurs:

- A previous invocation of the server that was connected to a client performed an active close, either
  by calling _close()_, or by crashing. This leaves a TCP endpoint that remains in the _TIME\_WAIT_
  state until the 2MSL timeout expires
- A previous invocation of the server created a child process to handle a connection to a client. Later,
  the server terminated, while the child continues to serve the client, and thus maintain a TCP endpoint
  using the server's well-known port

A connected TCP socket is identified by a 4-tuple of the following form:

```
{ local-IP-address, local-port, foreigin-IP-address, foreign-port }
```

However, most implementations enforce a stricter constraint: a local port can't be reused if any TCP
connection incarnation with a matching local port exists on the host. This rule is enforced even when
the TCP could not accept new connections.

Enabling the _SO\_REUSEADDR_ socket option relaxes this constraint, bringing it closer to the TCP
requirement. Setting the _SO\_REUSEADDR_ option means that we can bind a socket to a local port even
if another TCP is bound to the same port in either of the scenarios above.