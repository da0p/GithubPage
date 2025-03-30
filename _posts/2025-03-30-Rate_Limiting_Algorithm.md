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

## Leaking Bucket Algorithm

- The leaking bucket algorithm is similar to the token bucket except that
  requests are processed at a fixed rate. It is usually implemented with a FIFO
  queue.
- When a request arrives, the system checks if the queue is full. If it is not
  full, the request is added to the queue. Otherwise, the request is dropped.
  Requests are pulled from the queue and processed at regular intervals.
  ![Leaking Bucket](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/leaking_bucket.drawio.png)

Leaking bucket algorithm takes two parameters:

- Queue size: The queue holds the requests to be processed at a fixed rate.
- Outflow rate: it defines how many request can be processed at a fixed rate,
  usually in seconds.

| Pros                                                                                                               | Cons                                                    |
| ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------- |
| Memory efficient given the limited queue size                                                                      | A burst of traffic fills up the queue with old requests |
| Requests are processed at a fixed rate therefore it is suitable for use cases that a stable outflow rate is needed | Difficult to tune the parameters properly               |

## Fixed Window Counter

- The algorithm divides the timeline into fix-sized time windows and assign a
  counter for each window.
- Each request increments the counter by one.
- Once the counter reaches the pre-defined threshold, new requests are dropped
  until a new time window starts.
  ![Fixed Window Counter](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/fixed_window_counter.drawio.png)
- A major problem with this algorithm is that a burst of traffic at the edges of
  time windows could cause more requests than allowed quota to go through

| Pros                                                                              | Cons                                                                                                     |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Memory efficient                                                                  | Spike in traffic at the edges of a window could cause more requests than the allowed quote to go through |
| Easy to understand                                                                |                                                                                                          |
| Resetting available quota at the end of a unit time window fits certain use cases |

## Sliding Window Log

- In _Fixed Window Counter_, it allows more requests to go through at the edges
  of a window. The sliding window log algorithm fixes the issue.
- Sliding window log keeps track of request timestamps. Timestamp data is
  usually kept in cache, such as sorted sets of Redis.
- When a new request comes in, remove all the outdated timestamps. Outdated
  timestamps are defined as those older than the start of the current time
  window.
- Add timestamp of the new request to the log.
- If the log size is the same or lower than the allowed count, a request is
  accepted. Otherwise, it is rejected.

| Pros                                                           | Cons                                                                                                          |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| In any rolling window, requests will not exceed the rate limit | Consumes a lot of memory because even if a request is rejected, its timestamp might still be stored in memory |

## Sliding Window Counter

- A hybrid approach that combines the fixed window counter and sliding window
  log
- Formula to calculate the considered current number of requests:

$$requests_{current\_window} + requests_{previous\_window} * p$$

where p: overlap percentage of the rolling window and previous window

![Sliding Window Counter](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/sliding_window_counter.drawio.png)

| Pros                                                                                               | Cons                                                                                                                                                               |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Smooths out spikes in traffic because the rate is based on the average rate of the previous window | Only works for not-so-strict look back window. It is an approximation of the actual rate because it assumes requests in the previous window are evenly distributed |
| Memory efficient                                                                                   |                                                                                                                                                                    |
