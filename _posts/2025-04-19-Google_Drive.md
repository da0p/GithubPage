---
title: "Google Drive"
date: 2025-04-19
---

Google Drive is a file storage and synchronization service that helps you store documents, photos,
videos, and other files in the cloud

## Main Features

Features can be divided into: **functional requirements** and **non-functional requirements**

### Functional Requirements

- Add files
- Download files
- Sync files across multiple devices
- See file revisions
- Share files with your friends, family, and coworkers
- Send a notification when a file is edited, or shared with you

### Non-Functional Requirements

- _Reliability_: extremely important for a storage system. Data loss is unacceptable
- _Fast sync speed_: If file sync takes too much time, users will become impatient and abandon the product
- _Bandwidth usage_: If a product takes a lot of unnecessary network bandwidth, users will be unhappy,
  especially when they are on a mobile data plan
- _Scalability_: The system should be able to handle high volumes of traffic
- _High availability_: Users dhould still be able to use the system when some servers are offline, slowed down
  or have unexpected network errors

## System Design

![Google Drive Design](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/google_drive_design.drawio.png)

**Block servers:** block servers upload blocks to cloud storage. Block storage, refered to as block-level
storage, is a technology to store data files on cloud-based environments. A file can be split into several
blocks, each within a unique hash value, stored in our metadata database. Each block is treated as an independent
object and stored in our storage system (S3). To reconstruct a file, blocks are joined in a particular order. Block servers
can be optimized as the following to minimize the amount of network traffic being transmitted:

- _Delta sync_: When a file is modified, only modified blocks are synced instead of the whole file using a sync algorithm
- _Compression_: Applying compression on blocks can significantly reduce the data size. Thus, blocks are compressed using
compression algorithms depending on the file types.

**Cloud storage:** A file is split into smaller blocks and stored in cloud storage

**Cold storage:** Cold storage is a computer system designed for storing inactive data, meaning files are not
accessed for a long time

**Metadata database:** Some of the metadata are cached for fast retrieval

**Notification service:** It is a publisher/subscriber system that allows data to be transferred from notification
service to clients as certain events happen. In our specific case, notification service notifies relevant clients
when a file is added/edited/removed elsewhere so they can pull the latest changes

**Offline backup queue:** If a client is offline and cannot pull the latest file changes, the offline backup queue
stores the info so changes will be synced when the client is online