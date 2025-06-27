---
title: "Linux Series: Pseudoterminals"
date: 2025-06-02
---

A _pseudoterminal_ is a virtual device that provides an IPC channel. On one end of the channel is a
program that expects to be connected to a terminal device. On the other end is a program that drives
the terminal-oriented program by using the channel to send it input and read its output

## Overview

A pseudoterminal helps us to solve the problem: how can we enable a user on one host to operate a
terminal-oriented program on another host connected via a network?

![Pseudoterminals](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/pseudoterminals.drawio.png)

How _ssh_ uses a pseudoterminal

![SSH](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ssh.drawio.png)

## UNIX 98 Pseudoterminals

Various library functions used with UNIX 98 pseudoterminals:

- _posix\_openpt()_ function opens an unused pseudoterminal master device, returning a file descriptor
  that is used to refer to the device in later calls
- _grantpt()_ function changes the ownership and permissions of the slave device corresponding to a
  pseudoterminal master device
- _unlockpt()_ function unlocks the slave device corresponding to a pseudoterminal master device, so
  that the slave device can be opened
- _ptsname()_ function returns the name of the slave device corresponding to a pseudoterminal master
  device. The slave device can then be opened using _open()_