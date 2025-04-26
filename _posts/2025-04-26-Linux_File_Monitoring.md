---
title: "Linux Series: Monitoring File Events"
date: 2025-04-26
---

Some applications need to be able to monitor files or directories in order to determing whether
events have occurred for the monitored objects

## Usage of _inotify_ API

![Usage of inotify](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/inotify_usage.drawio.png)

## _inotify_ Events

| **Bit value**        | **In**     | **Out**     | **Description**                                       |
| -------------------- | ---------- | ----------- | ----------------------------------------------------- |
| IN_ACCESS            | y          | y           | File was accessed (read())                            |
| IN_ATTRIB            | y          | y           | File metadata changed                                 |
| IN_CLOSE_WRITE       | y          | y           | File opened for writing was closed                    |
| IN_CLOSE_NOWRITE     | y          | y           | File was opened read-only was closed                  |
| IN_CREATE            | y          | y           | file /directory created inside watched directory      |
| IN_DELETE            | y          | y           | File /directory deleted from within watched directory |
| IN_DELETE_SELF       | y          | y           | Watched file /directory was itself deleted            |
| IN_MODIFY            | y          | y           | File was modified                                     |
| IN_MOVE_SELF         | y          | y           | Watched file /directory was itself moved              |
| IN_MOVED_FROM        | y          | y           | File moved out of watched directory                   |
| IN_MOVED_TO          | y          | y           | File moved into watched directory                     |
| IN_OPEN              | y          | y           | File was opened                                       |
| ------------------   | --------   | ---------   | ----------------------------------------              |
| IN_ALL_EVENTS        | y          | n           | Shorthand for all of the above input events           |
| IN_MOVE              | y          | n           | Shorthand for IN_MOVED_FROM \| IN_MOVED_TO            |
| IN_CLOSE             | y          | n           | Shorthand for IN_CLOSE_WRITE \| IN_CLOSE_NOWRITE      |
| -------------------- | ---------- | ----------- | -------------------------                             |
| IN_DONT_FOLLOW       | y          | n           | Don't dereference symbolic link                       |
| IN_MASK_ADD          | y          | n           | Add events to current watch mask for _pathname_       |
| IN_ONESHOT           | y          | n           | Monitor _pathname_ for just one event                 |
| IN_ONLYDIR           | y          | n           | Fail if _pathname_ is not a directory                 |
| --                   | --         | --          | --                                                    |
| IN_IGNORED           | n          | y           | Watch was removed by application or by kernel         |
| IN_ISIDR             | n          | y           | Filename returned in _name_ is a directory            |
| IN_Q_OVERFLOW        | n          | y           | Overflow on event queue                               |
| IN_UNMOUNT           | n          | y           | File system containing object was unmounted           |

_inotify_ event has the following structure

```C
struct inotify_event {
    int wd;  /* watch descriptor on which event occurred */
    uint32_t mask; /* Bits describing event that occurred */
    uint32_t cookie; /* Cookie for related events (for rename()) */
    uint32_t len; /* Size of 'name' field */
    char name[]; /* Optional null-terminated filename */
}
```

and the buffer when using _read()_ can have the following structure

![inotify event](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/inotify_event.drawio.png)

## Example

```cpp
#include <cstdlib>
#include <iostream>
#include <climits>
#include <linux/limits.h>
#include <string>
#include <map>

#include <sys/inotify.h>
#include <unistd.h>

namespace {
    // Allocate enough memory for 10 events
    constexpr uint32_t gBufLen = 10 * (sizeof(inotify_event) + NAME_MAX + 1);

    std::string
    event2Str(uint32_t mask)
    {
        std::map<uint32_t, std::string> event2StrTable {
            {IN_ACCESS, "IN_ACCESS"},
            {IN_ATTRIB, "IN_ATTRIB"},
            {IN_CLOSE_NOWRITE, "IN_CLOSE_NOWRITE"},
            {IN_CLOSE_WRITE, "IN_CLOSE_WRITE"},
            {IN_CREATE, "IN_CREATE"},
            {IN_DELETE, "IN_DELETE"},
            {IN_DELETE_SELF, "IN_DELETE_SELF"},
            {IN_IGNORED, "IN_IGNORED"},
            {IN_ISDIR, "IN_ISDIR"},
            {IN_MODIFY, "IN_MODIFY"},
            {IN_MOVE_SELF, "IN_MOVE_SELF"},
            {IN_MOVED_FROM, "IN_MOVED_FROM"},
            {IN_MOVED_TO, "IN_MOVED_TO"},
            {IN_OPEN, "IN_OPEN"},
            {IN_Q_OVERFLOW, "IN_Q_OVERFLOW"},
            {IN_UNMOUNT, "IN_UNMOUNT"}
        };

        std::string eventStr;
        uint32_t shift = 0;
        while (mask != 0) {
            if (mask & 0x1) {
                auto event = 0x1 << shift;
                if (event2StrTable.count(event) != 0) {
                    eventStr += event2StrTable.at(event);
                }
            }
            mask = mask >> 1;
            shift++;
        }
        return eventStr;
    }

    void
    displayInotifyEvent(inotify_event* i)
    {
        std::cout << "  wd = " << i->wd << "; ";
        if (i->cookie > 0) {
            std::cout << "cookie = " << i->cookie << "; ";
        }
        std::cout << "mask = " << event2Str(i->mask) << "\n";
        if (i->len > 0) {
            std::cout << "   name = " << i->name << "\n";
        }
    }

    void usageErr(const std::string& progName) {
        std::cout << "Usage:\n";
        std::cout << progName << " pathname... \n";
        exit(EXIT_FAILURE);
    }

    void errExit(const std::string& reason) {
        std::cout << "Error in " << reason;
        exit(EXIT_FAILURE);
    }

    void fatal(const std::string& reason) {
        std::cout << reason << "\n";
        exit(EXIT_FAILURE);
    }
}

int main(int argc, char *argv[])
{
    if (argc < 2 || std::string(argv[1]) == "--help") {
        usageErr(argv[0]);
    }

    // Create inotify instance
    auto inotifyFd = inotify_init();
    if (inotifyFd == -1) {
        errExit("inotify_init");
    }

    for (int i = 0; i < argc; i++) {
        // Add watch descriptor for all events for files specified by argv[i]
        auto wd = inotify_add_watch(inotifyFd, argv[i], IN_ALL_EVENTS) ;       
        if (wd == -1) {
            errExit("inotify_add_watch");
        }
    }

    char buf[gBufLen];
    while (true) {
        // Read from the inotify instance
        auto numRead = read(inotifyFd, buf, gBufLen);
        if (numRead == 0) {
            fatal("read() from inotify fd returned 0!");
        }

        if (numRead == -1) {
            errExit("read");
        }

        std::cout << "Read " << (long) numRead << " bytes from inotify fd\n";
        for (auto p = buf; p < buf + numRead; ) {
            auto *event = reinterpret_cast<inotify_event*>(p);
            displayInotifyEvent(event);
            p += sizeof(inotify_event) + event->len;
        }
    }

    exit(EXIT_SUCCESS);
}
```