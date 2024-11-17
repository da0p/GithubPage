---
title: "NTP Time Synchronization"
date: 2024-09-15
---

# How NTP works?

The following diagram captures how Network Time Protocol works in a nutshell

![NTP](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/NTP.drawio.png)

# What to use

![Chrony](https://chrony-project.org/) can be used to synchronize time with
different configurations. In a distributed system with only one node that can
communicate with a NTP server, this node can become the NTP server for other
clients in the local network.

![DistributedSystem](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/NTP_Distributed.drawio.png)
