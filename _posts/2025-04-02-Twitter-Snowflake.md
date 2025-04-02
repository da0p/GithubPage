---
title: "Twitter Snowflake"
date: 2025-04-02
---

In order to generate unique IDs in distributed systems, multiple options can be
used:

- Multi-master replication
- Universally unique identifier (UUID)
- Ticket server
- Twitter snowflake

## Multi-master Replication

The approach uses the databases' _auto increment_ feature. Instead of increasing
the next ID by 1, we increase it by k, where k is the number of database servers
in use.

![Multi-master Replication](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/multi_master_replication.drawio.png)

However, there are some major drawbacks:

- Hard to scale with multiple data centers
- IDs do not go up with time across multiple servers
- It does not scale well when a server is added or removed

## UUID

UUID is an easy way to obtain unique IDs. UUID is a 128-bit number used in
information in computer systems. UUID has a very low probability of getting
collisions.

| Pros                                                                                              | Cons                         |
| ------------------------------------------------------------------------------------------------- | ---------------------------- |
| - Generating UUID is simple. No coordination between servers is needed, no synchronization issues | - IDs are 128-bit long       |
| - Easy to scale                                                                                   | - IDs do not go up with time |
|                                                                                                   | IDs could be non-numeric     |

## Ticket Server

The idea is to use a centralized _auto increment_ feature in a single database
server.

| Pros                                                      | Cons                                                                                      |
| --------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Numeric IDs                                               | Single point of failure, can set up multiple ticket servers, but difficult to synchronize |
| Easy to implement, works for small to medium applications |                                                                                           |

## Twitter Snowflake

Twitter uses the following layout for 64-bit ID

![Twitter Snowflake](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/twitter_snowflake.drawio.png)

- **Sign bit**: 1 bit. It will always be 0. This is reserved for future use. It
  can potentially be used to distinguish between signed and unsigned numbers.
- **Timestamp**: 41 bits. Milliseconds since the epoch or custom epoch.
- **Datacenter ID**: 5 bits or 32 datacenters
- **Machine ID**: 5 bits or 32 machines per data center
- **Sequence number**: 12 bits. Supports 4096 new IDs per milliseconds. This
  number is reset to 0 every millisecond.
