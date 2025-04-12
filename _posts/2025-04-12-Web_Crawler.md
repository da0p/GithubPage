---
title: "Web Crawler"
date: 2025-04-12
---
A web crawler does the following basic algorithm:

- Given a set of URLs, download all the web pages addressed by the URLs
- Extract URLs from these web pages
- Add new URLs to the list of URLs to be downloaded

Good characteristics of a web crawler:

- **Scalability**: Web crawling should be efficient using parallelization
- **Robustness**: Web crawler must be able to handle errors like bad HTML, 
unresponsive servers, crashes, malicious links
- **Politeness**: The crawler should not make too many requests to a website
within a short time interval
- **Extensibility**: It must be flexible to support new content types

## Web Crawler Design

![Web Crawler](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/web_crawler.drawio.png)

**Seed URLs**: A web crawler uses seed URLs as a starting point for the crawl
process. There are different ways to choose seed urls:

- Based on locality as different countries may have different popular websites 
- Based on topics

**URL Frontier**: Most modern web crawlers split the crawl state into two

- To be downloaded
- Already downloaded

**HTML Downloader**: downloads web pages from the internet. Those URLs are
provided by the the **URL Frontier**

**DNS Resolver**: To download a web page, a URL must be translated into an IP
address.
The **HTML Downloader** calls the **DNS Resolver** to get the corresponding IP
address for the URL

**Content Parser**: After a web page is downloaded, the content must be parsed.
If this parser stays in the web crawler, it will slow down the crawling process

**Content Seen?**: In order to know whether the content of the web page is not
duplicated, we can hash the content and only need compare the hashes

**Content Storage**: It is a storage system for storing HTML content. The
choice of storage system depends on factors such as data type, data size, access
frequency, life span. Both disk and memory are used.

- Most of the content is stored on disk because the data set is too big to fit
in memory
- Popular content is kept in memory to reduce latency

**URL Extractor**: extracts links from HTML pages

**URL Filter**: excludes certain content types, file extensions, error links
and URLs in blacklist

**URL Seen?**: a data structure that keeps track of URLs that are visited before
or already in the **URL Frontier**. It helps to avoid adding the same URL
multiple times as this can increase server load and cause potential infinite
loops

**URL Storage**: stores already visited URLs

### Politeness

The component that stores URLs to be downloaded is called the URL Frontier. It
ensures politeness, prioritization, and freshness.

In order to ensure prioritization, the prioritizer takes the input URLs and
computes the priorities. Queue f1 to fn has an assigned priority. Queues with
high priority are selected with higher probablity

In order to ensure politeness, a mapping from website hostnames to download
threads. Ech downloader thread has a separate FIFO queue and only downloads URLs
obtained from queue. It ensures that each queue (b1, b2, ..., bn) only contains
URLs from the same host

In order to ensure freshness, a web crawler must periodically recrawl downloaded
pages to keep the data set fresh. However, it should

- Recrawl based on web pages' update history
- Prioritize URLs and recrawl important pages first and more frequently

### Robustness

In order to improve the system robustness

- **Consitent Hashing**: This helps to distribute loads among downloaders. A new
downloader server can be added or removed using consistent hashing
- **Save crawl states and data**: To guard against failures, crawl states and data
are written to a storage system. A disrupted crawl can be restarted easily by
loading saved states and data
- **Exception handling**: Errors are inevitable and common in a large-scale
system. The crawler must handle exceptions gracefully without crashing the system
- **Data validation**: an important measure to prevent system errors

### Extensibility

The crawler can be extended by plugging in new modules

- **PNG Downloader**: a module is plugged-in to download PNG files
- **Web Monitor**: a module to monitor the web and prevent copyright and
trademark infringements