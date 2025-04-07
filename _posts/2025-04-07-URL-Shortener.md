---
title: "URL Shortener"
date: 2025-04-07
---

When we enter a short url, we will visit the server that contains the mapping
from the short url to long url. The server will responds with 301 code. The
client will visit the long URL.

![URL Shortener](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/url_shortener.drawio.png)

How do we do the mapping?

## Hash and Collision Resolution

To shorten a long URL, a hash function that hashes a long URL to a shorter
string should be implemented. Common hash functions:

- CRC32 (4 bytes)
- MD5 (16 bytes)
- SHA-1 (20 bytes)

If we want a shorter URL, like 7-character long. We can do:

- Collect only the first 7 characters of a hash value
- If there is a collision, we can recursively append a new predefined string
  until now more collision discovered.

This method can eliminate collision, however, it is expensive to query the
database to check if a short URL, exists for every request. A technique called
bloom filters can improve performance. A bloom filter is a space-efficient
probabilistic technique to test if an element is a member of a set.

## Base 62 Conversion

Base 62 conversion is another approach used for URL shorteners. Base conversion
helps to convert the same number between its different number representation
systems. Base 62 conversion is used as there are 62 possible characters for hash
values.

![Base 62](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/base62.drawio.png)

## Comparison

| Hash and Collision Resolution                                                          | Base 62 Conversion                                                                                               |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Fixed short URL length                                                                 | The short URL length is not fixed. It goes up with the ID                                                        |
| No need a unique ID generator                                                          | Depends on a unique ID generator                                                                                 |
| Collision is possible and must be resolved                                             | Collision is impossible because ID is unique                                                                     |
| Impossible to figure out the next available short URL because it does not depend on ID | Easy to figure out the next available short URL if ID increments by 1 for a new entry. Can be a security concern |
