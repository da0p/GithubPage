---
title: "Consistent Hashing"
date: 2025-03-30
---

Consistent hashing is a commonly used technique to distribute requests/data
efficiently and evenly across servers.

## Naive Hashing Approach

If we have n cache servers, then we can balance the load by hasing

$$serverIndex = hash(key) \% N $$

where N is the size of the server pool

However, this approach only works well when the number of servers does not
change. If one server is down (removed) or added, it will trigger the
redistribution of keys and consequently a lot of cache misses.

![Basic Hashing](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/basic_hashing.drawio.png)

## Consistent Hashing Approach

In order to solve the problem described with the naive hashing approach,
consisten hashing is used. Consistent hashing is a special kind of hashing such
that when a hash table is re-sized and consistent hashing is used, only k/n keys
need to be remapped on average, where k is the number of keys, and n is the
number of slots.

![Consistent Hashing](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/consistent_hashing.drawio.png)

With this approach, whenever a server is added or removed from the pool, then we
just need to go counter-clockwise from the removed/added server until another
server is reached and redistribute only the keys in this partition to the
correct server.

![Consistent Hashing](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/consistent_hashing_removal.drawio.png)

However, there are still two flaws in this approach

- Impossible to keep the same size of partitions on the ring for servers
  considering a server can be added or removed
- It is possible to have non-uniform key distribution on the ring causing
  unbalanced load on one specific server

In order to solve these issues, a technique called virtual nodes or replicas is
used. As the number of virtual nodes increases, the distribution of keys becomes
more balanced. However, more spaces are needed to store data about virtual
nodes.

![Virtual Nodes](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/consistent_hashing_virtual_nodes.drawio.png)
