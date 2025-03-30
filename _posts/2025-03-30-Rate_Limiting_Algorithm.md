---
title: "Rate Limiting Algorithms"
date: 2025-03-30
---

Rate limiting can be implemented using different algorithms, and each of them
has pros and cons.

## Token Bucket

- A token bucket is a container that has pre-defined capacity. Tokens are put in
  the bucket at preset rates periodically. Once the bucket is full, no more
  tokens are added. Once the bucket is full, extra tokens will overflow.
- Each request consumes one token. When a request arrives, we check if there are
  enough tokens in the bucket. If there are enough tokens, we take one token out
  for each request, and the request goes through. If there are not enough
  tokens, the request is dropped.
  ![Token Bucket](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/token_bucket.drawio.png)

The token bucket algorithm takes two parameters:

- Bucket size: the maximum number of tokens allowed in the bucket
- Refill rate: number of tokens put into the bucket every second

| Pros                                                                                                   | Cons                                   |
| ------------------------------------------------------------------------------------------------------ | -------------------------------------- |
| Easy to implment                                                                                       | Difficult to tune two parameters right |
| Memory efficient                                                                                       |                                        |
| Allows a burst of traffic for short periods. A request can go through as long as there are tokens left |                                        |
