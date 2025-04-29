---
title: "Linux Series: Signals"
date: 2025-04-27
---

## Concepts

A _signal_ is a notification to a process that an event has occurred. Signals are sometimes called
_software interrupts_ 

The usual source of many signals sent to a process is the kernel. It can be due to the following causes:

- A hardware exception occurred
- User types one of the terminal special characters that generate signals. These characters include
the _interrupt_ character
- A software event occurred

Two categories for signals:

- _Traditional_ or _standard_ signals
- _Realtime_ signals

A signal is said to be _generated_ by some event. Once generated, a signal is later _delivered_ to a
process, which then takes some action in response to the signal. Between the time it is generated and
the time it is delivered, a signal is said to be _pending_

A pending signal is delivered to a process as soon as it is next scheduled to run, or immediately
if the process is already running. However, sometimes, we need to ensure a segment of code is not
interrupted by the delivery of a signal, we can do that by adding a signal to the process's
_signal mask_-a set of signals whose delivery is currently _blocked_. If a signal is generated while
it is blocked, it remains pending until it is later unblocked.

When a process receives a signal, it can decides what to do with it:

- Ignore
- Terminate
- Create a core dump file
- Stop
- Resume

A process can change the default behavior of handling a signal by registering a signal handler

## Linux Signals and Default Actions

| **Name**      | **Signal number**      | **Description**             | **SUSv3** | **Default** |
| ------------- | ---------------------- | --------------------------- | --------- | ----------- |
| SIGABORT      | 6                      | Abort process               | y         | core        |
| SIGALRM       | 14                     | Real-time timer expired     | y         | term        |
| SIGBUS        | 7 (SAMP=10)            | Memory access error         | y         | core        |
| SIGCHLD       | 17 (SA=20, MP=18)      | Child terminated or stopped | y         | ignore      |
| SIGCONT       | 18 (SA=19, M=25, P=26) | Continue if stopped         | y         | cont        |
| SIGEMT        | undef (SAMP=7)         | Hardware fault              | n         | term        |
| SIGFPE        | 8                      | Arithmetic exception        | y         | core        |
| SIGHUP        | 1                      | Hangup                      | y         | term        |
| SIGILL        | 4                      | Illegal instruction         | y         | core        |
| SIGINT        | 2                      | Terminal interrupt          | y         | term        |
| SIGIO/SIGPOLL | 29 (SA=23, MP=22)      | I/O possible                | y         | term        |
| SIGKILL       | 9                      | Sure kill                   | y         | term        |
| SIGPIPE       | 13                     | Broken pipe                 | y         | term        |
| SIGPROF       | 27 (M=29, P=21)        | Profiling timer expired     | y         | term        |
| SIGPWR        | 30 (SA=29, MP=19)      | Power about to fail         | n         | term        |
| SIGQUIT       | 3                      | Terminal quit               | y         | core        |
| SIGSEGV       | 11                     | Invalid memory reference    | y         | core        |
| SIGSTKFLT     | 16 (SAM=undef, P=36)   | Stack fault on coprocessor  | n         | term        |
| SIGSTOP       | 19 (SA=17, M=23, P=24) | Sure stop                   | y         | stop        |
| SIGSYS        | 31 (SAMP=12)           | Invalid system call         | y         | core        |
| SIGTERM       | 15                     | Terminate process           | y         | term        |
| SIGTRAP       | 5                      | Trace/brakepoint trap       | y         | core        |
| SIGTSTP       | 20 (SA=18, M=24, P=25) | Terminal stop               | y         | stop        |
| SIGTTIN       | 21 (M=26, P=27)        | Terminal read from BG       | y         | stop        |
| SIGTTOU       | 22 (M=27, P=28)        | Terminal write from BG      | y         | stop        |
| SIGURG        | 23 (SA=16, M=21, P=29) | Urgent data on socket       | y         | ignore      |
| SIGUSR1       | 10 (SA=30, MP=16)      | User-defined signal 1       | y         | term        |
| SIGUSR2       | 12 (SA=31, MP=17)      | User-defined signal 2       | y         | term        |
| SIGVTALRM     | 26 (M=28, P=20)        | Virtual timer expired       | y         | term        |
| SIGWINCH      | 28 (M=20, P=23)        | Terminal window size change | n         | ignore      |
| SIGXCPU       | 24 (M=30, P=33)        | CPU time limit exceeded     | y         | core        |
| SIGXFSZ       | 25 (M=31, P=34)        | File size limit exceeded    | y         | core        |

There are two ways to change the disposition of a signal:

- _signal(...)_
- _sigaction(...)_

_sigaction(...)_ is the preferable way since it is portable

## Signal Handlers

A _signal handler_ is a function that is called when a specified signal is delivered to a process

![Signal Handler Execution](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/signal_handler_execution.drawio.png)

## Sending Signals

One process can send a signal to another process using the _kill()_ system call, which is the analog
of the _kill_ shell command

```C
#include <signal.h>

int kill(pid_t pid, int sig);
```

Four different cases determine how _pid_ is interpreted:

- If _pid_ is greater than 0, the signal is sent to the process with the process ID specified by _pid_
- If _pid_ equals 0, the signal is sent to every process in the same process group as the calling process,
including the calling process itself
- If _pid_ is less than -1, the signal is sent to all of the processes in the process group whose ID
equals the absolute value of _pid_. Sending a signal to all of the processes in a process group finds
particular use in shell job control
- If _pid_ equals -1, the signal is sent to every process for which the calling process has permission
to send a signal, except _init_ and the calling process. If a privileged process makes this call, then
all processes on the system will be signaled, except for these last two

_kill()_ can be used to check the existence of a process when the _sig_ argument is specified as 0.

```cpp
#include <cstdlib>
#include <iostream>
#include <string>

#include <signal.h>

void usageError(const std::string& funcName)
{
    std::cout << funcName << " sig-num pid\n";
    exit(EXIT_FAILURE);
}

void errExit(const std::string& funcName)
{
    std::cout << "Error in using " << funcName << "\n";
    exit(EXIT_FAILURE);
}

int main(int argc, char *argv[])
{
    if (argc != 3 || std::string(argv[1]) == "--help") {
        usageError(argv[0]);
    }

    auto sig = std::stoi(argv[1]);
    auto s = kill(std::stol(argv[2]), sig);

    if (sig != 0) {
        if (s == -1) {
            errExit("kill-1");
        }
    } else {
        if (s == 0) {
            std::cout << "Process exists and we can send it a signal\n";
        } else {
            if (errno == EPERM) {
                std::cout << "Process exists, but we don't have permission to send it a signal\n";
            } else if (errno == ESRCH) {
                std::cout << "Process does not exist\n";
            } else {
                errExit("kill-2");
            }
        }
    }
}
```

## Signal Sets

Many signal-related system calls need to be able to represent a group of different signals. For
instance, _sigaction()_ and _sigprocmask()_ allow a program to specify a group of signals that are
to be blocked by a process, while _sigpending()_ returns a group of signals that are currently
pending for a process. Multiple signals are represented using a data structure called a _signal set_,
provided by the system data type _sigset\_t_

## Signal Mask

For each process, the kernel maintains a _signal mask_-a set of signals whose delivery to the process
is currently blocked. If a signal that is blocked is sent to a process, delivery of that signal is
delayed until it is unblocked by being removed from the process signal mask.
A signal may be added to the signal mask in the following ways:

- When a signal handler is invoked, the signal that caused its invocation can be automatically added
to the signal mask. Whether or not this occurs depends on the flags used when the handler is
established using _sigaction()_
- When a signal handler is established with _sigaction()_, it is possible to specify an additional
set of signals that are to be blocked when the handler is invoked
- The _sigprocmask()_ system call can be used at any time to explicitly add signals to, and remove
signals from, the signal mask

## Pending Signals

If a process receives a signal that it is currently blocking, that signal is added to the process's
set of pending signals. When the signal is later unblocked, it is then delivered to the process. We
can use _sigpending()_ to determine which signals are pending for a process

The set of pending signals is only a maks; it indicates whether or not a signal has occurred, but
not how many times it has occurred. In other words, if the same signal is generated multiple times
while it is blocked, then it is recorded in the set of pending signals, and later delivered, just
once.

## Changing Signal Dispositions

We can use _sigaction()_ system call instead of _signal()_ for setting the disposition of a signal.
_sigaction()_ is more complex but provides greater flexibility

```C
#include <signal.h>

int sigaction(int sig, const struct sigaction* act, struct sigaction *oldact);
```

- _sig_: the signal whose disposition want want to retrieve or change. Can be anything except
_SIGKILL_ or _SIGSTOP_
- _act_: a pointer to a structure specifying a new disposition for the signal. If we want to find
existing disposition of the signal, then we can specify _NULL_ for this argument. The _oldact_
argument is a pointer to a structure of the same type, and is used to return information about the
signal's previous disposition. If we are not interested in this information, we can set it to _NULL_

```C
struct sigaction {
    void (*sa_handler)(int);    /* Address of handler */
    sigset_t sa_mask;           /* Signals blocked during handler invocation */
    int sa_flags;               /* Flags controlling handler invocation */
    void (*sa_restorer)(void);  /* Not for application use */
}
```

- _sa\_handler_ specifies the signal handler address
- _sa\_mask_ defines a set of signals that are to be blocked during invocation of the handler
defined by _sa\_handler_. When the signal handler is invoked, any signals in this set that are not
currently part of the process signal mask are automatically added to the mask before the handler is
called. These signals remain in the process signal mask until the signal handler returns, at which
time they are automatically removed. The _sa\_mask_ field allows us to specify a set of signals that
aren't permitted to interrupt execution of this handler. Besides, the signal that caused the
invocation is added to the process signal mask automatically. This will avoid the signal handler
recursively interrupts itself if a second instance of the same signal arrives while the handler is
executing. Because blocked signals are not queued, if any of these signals are repeatedly generated
during the execution of the handler, they are (later) delivered only once.
- _sa\_flags_ is a bit mask specifying various options controlling how the signal is handled

```cpp
#include <array>
#include <bits/types/sig_atomic_t.h>
#include <chrono>
#include <csignal>
#include <cstdint>
#include <cstdlib>
#include <iostream>

#include <signal.h>
#include <string>
#include <thread>

class SigReceiver {
public:
  SigReceiver() { setup(); }

  void run(const std::string &prog, std::chrono::seconds sleepTime) {
    std::cout << prog << ": PID is " << getpid() << "\n";

    if (sleepTime != std::chrono::seconds(0)) {
      sigset_t blockingMask;
      sigfillset(&blockingMask);
      if (sigprocmask(SIG_SETMASK, &blockingMask, NULL) == -1) {
        errExit("sigprocmask");
      }

      std::cout << prog << " sleeping for " << sleepTime.count()
                << " seconds\n";
      std::this_thread::sleep_for(sleepTime);

      sigset_t pendingMask;
      if (sigpending(&pendingMask) == -1) {
        errExit("sigpending");
      }

      sigset_t emptyMask;
      sigemptyset(&emptyMask);
      if (sigprocmask(SIG_SETMASK, &emptyMask, NULL) == -1) {
        errExit("sigprocmask");
      }
    }

    while (!mGotSigInt) {
      continue;
    }

    for (int i = 1; i < NSIG; i++) {
      if (mSigCnt[i] != 0) {
        std::cout << prog << ": signal " << i << " caught " << mSigCnt[i]
                  << " time\n";
      }
    }
  }

  void setup() {
    struct sigaction callback;
    callback.sa_handler = handler;
    for (auto n = 1; n < NSIG; n++) {
      sigaction(n, &callback, nullptr);
    }
  }

private:
  static void handler(int sig) {
    if (sig == SIGINT) {
      mGotSigInt = 1;
    } else {
      mSigCnt[sig]++;
    }
  }

  void errExit(const std::string &funcName) {
    std::cout << "Error on using " << funcName << "\n";
    exit(EXIT_FAILURE);
  }

  static volatile sig_atomic_t mGotSigInt;
  static std::array<uint32_t, NSIG> mSigCnt;
};

volatile sig_atomic_t SigReceiver::mGotSigInt = 0;
std::array<uint32_t, NSIG> SigReceiver::mSigCnt;

int main(int argc, char *argv[]) {
  auto sleepTime = 0;
  if (argc > 1) {
    sleepTime = std::stol(argv[1]);
  }

  SigReceiver sigReceiver;

  sigReceiver.run(argv[0], std::chrono::seconds(sleepTime));

  return 0;
}
```
