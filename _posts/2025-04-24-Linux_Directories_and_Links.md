---
title: "Linux Series: Directories and Links"
date: 2025-04-25
---

Each process has two directory-related attributes:

- A root directory, which determines the point from which absolute pathnames are interpreted
- A current working directory, which determines the point from which relative pathnames are interpreted

## Directories and Links

A _directory_ is stored in the file system in a similar way to a regular file. Two things distinguish
a directory from a regular file:

- A directory is marked with a different file type in its i-node entry
- A directory is a file with a special organization. It is a table consisting of filenames and
i-node numbers

![Relationship between i-node and directory structures](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/i_node_and_directory.drawio.png)

_i\-node_ doesn't contain a filename; it is only the mapping within a directory list that defines
the name of a file. Hence, we can create multiple names - in the same or in different directories -
each of which refers to the same i-node. These multiple names are known as _links_, or sometimes as
_hard links_ to distinguish them from symbolic links.

Two limitations of hard links:

- Because directory entries (hard links) refer to files using just an i-node number, and i-node numbers
are unique only within a file system, a hard link must reside on the same file system as the file to
which it refers
- A hard link can't be made to a directory. This prevents the creation of circular links, which could
confuse many system programs

## Symbolic (Soft) Links

A _symbolic link_, also sometimes called a _soft link_, is a special file type whose data is the name
of another file. The pathname to which a symbolic link refers maybe either absolute or relative. A
relative symbolic link is interpreted relative to the location of the link itself

Symbolic links don't have the same status as hard links. In particular, a symbolic link is not included
in the link count of the file to which it refers. Therefore, if the filename to which the symbolic refers
is removed, the symbolic link itself continues to exist, even though it can no longer be dereferenced.
This link is called a _dangling link_

![Relationship between i-node and soft links](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/i_node_and_soft_link.drawio.png)

Since a symbolic link refers to a filename, rather than an i-node number, it can be used to link to
a file in a different file system. Symbolic links also do not suffer the other limitation of hard links:
we can create soft links to directories