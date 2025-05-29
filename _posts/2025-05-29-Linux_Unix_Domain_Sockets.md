---
title: "Linux Series: Unix Domain Sockets"
date: 2025-05-29
---

## UNIX Domain Socket Addresses: _struct sockaddr\_un_

In the UNIX domain, a socket address takes the form of a pathname, and the domain-specific socket
address structure is defined as follows:

```c
struct sockaddr_un {
    sa_family_t sun_family;     /* Always AF_UNIX */
    char sun_path[108];         /* Null-terminated socket pathname */
};
```

Some notes about binding a UNIX domain socket:

- When used to bind a UNIX domain socket, _bind()_ creates an entry in the file system. The ownership
  of the file is determined according to the usual rules for file creation. The file is marked as a
  socket
- We can't bind a socket to an existing pathname (_bind()_ fails with the error _EADDRINUSE_)
- It is usual to bind a socket to an absolute pathname, so that the socket resides at a fixed address
  in the file system. Using a relative pathname is possible, but unusual, because it requires an application
  that wants to _connect()_ to this socket to know the current working directory of the application
  that performs the _bind()_
- A socket may be bound to only one pathname; conversely, a pathname can be bound to only one socket
- We can't use _open()_ to open a socket
- When the socket is no longer required, its pathname entry can (and generally should) be removed using
  _unlink()_ (or _remove()_)
- For UNIX domain sockets, datagram transmission is carried out within the kernel, and is reliable. All
  messages are delivered in order and unduplicated

In most of our example programs, we bind UNIX domain sockets to pathnames in the /tmp directory, because
this directory is normally present and writable on every system. Real-world applications should _bind()_
UNIX domain sockets to absolute pathnames in suitably secured directories

## UNIX Domain Socket Permissions

The ownership and permissions of the socket file determine which processes are able to communicate with
that socket:

- To connect to a UNIX domain stream socket, write permission is required on the socket file
- To send a datagram to a UNIX domain datagram socket, write permission is required on the socket file
- Execute (search) permission is required on each of the directories in the socket pathname

## Linux Abstract Socket Namespace

The so-called _abstract namespace_ is a Linux-specific feature that allows us to bind a UNIX domain
socket to a name without that name being created in the file system. This provides a few potential
advantages:

- No possible collisions with existing names in the file system
- No need to unlink the socket pathname after using the socket. It is automatically removed when the
  socket is closed
- No need to create a file-system pathname for the socket

To create an abstract binding, we specify the first byte of the _sun\_path_ filed as a null byte (\0).
This distinguishes abstract socket names from conventional UNIX domain socket pathnames, which consist
of a string of one or more nonnull bytes terminated by a null byte. The remaining bytes of the _sun\_path_
field then define the abstract name for the socket

```c
struct sockaddr_un addr;

// clear address structure
memset(&addr, 0, sizeof(struct sockaddr_un));
addr.sun_family = AF_UNIX;

// addr.sun_path[0] has already been set to 0 by memset()
// abstract name is "xyz" followed by null bytes
strncpy(&addr.sun_path[1], "xyz", sizeof(addr.sun_path) - 2);

sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
if (sockfd == -1) {
    exit(EXIT_FAILURE);
}
if (bind(sockfd, (struct sockaddr *) &addr, sizeof(struct sockaddr_un)) == -1) {
    exit(EXIT_FAILURE);
}
```