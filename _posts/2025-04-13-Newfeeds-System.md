---
title: "News Feed System"
date: 2025-04-13
---

News feed is the constantly updating list of stories in the middle of your home
page.

## System Design

There are two flows in this sytem:

- **Feed publishing**: When a user publishes a post, the corresponding data is
written into cache and database. A post is then populated to her friends news
feed
- **Newsfeed building**: Newsfeed can be built by aggregating friends' posts in
reverse chronological order

### Newsfeed APIs

Since there are two flows, we also need two APIs:

- Feed publishing API
- News feed retrieval API

#### Feed Publishing API

In order to publish a post, a HTTP POST request will be sent to the server. The
API can be: _POST/v1/me/feed_ with the following parameters:

- _content_: content is the text of the post
- _auth\_token_: used to authentiate API requests

#### Newsfeed Retrieval API

The API can be _GET/v1/me/feed_ with the following parameters:

- _auth\_token_: used to authenticate API requests

### Feed Publishing Design

![Feed Publishing](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/feed_publising.drawio.png)

Fanout is the process of delivering a post to all friends. There are two types
of fanout models:

- **Fanout on write (push model)**: news feed is pre-computed during write time. A new post
is delivered to friends' cache immediately after it is published
- **Fanout on read (pull model)**: news feed is generated during read time. This is an
on-demand model. Recent posts are pulled when a user loads his/her home page

| Model               | Pros                                                                                                                               | Cons                                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Fanout on write** | - News feed is generated in real-time and can be pushed to friends immediately                                                     | - Hotkey problem when a user has many friends, fetching friend list and generating news feed for all are slow |
|                     | - Fetching news feed is fast because the news feed pre-computed during write time                                                  | - For inactive users or those rarely log in, pre-computing news feeds waste computing resources               |
| **Fanout on read**  | - For inactive users or those who rarely log in, fanout on read works better because it will not waste computing resources to them | - Getching the news feed is slow as the news feed is not pre-computed                                         |
|                     | - Data is not pushed to friends so there is no hotkey problem                                                                      |                                                                                                               |

If we use a hybrid approach, use push model for the majority of users, and use
pull model for celebrities or users who have many friends/followers. The
followers will pull news content on-demand to avoid system overload

### Newsfeed Retrieval Design

![Feed Retrieval](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/feed_retrieval.drawio.png)

### Cache Architecture

![Cache Architecture](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/cache_architecture.drawio.png)

- _News Feed_: it stores iDs of news feeds
- _Content_: it stores every post data. Popular content is stored in hot cache
- _Social Graph_: it stores user relationship data
- _Action_: it stores info about whether a user liked a post, replied a post, or took other actions on a post
- _Counters_: it stores counters for like, reply, follower, following
