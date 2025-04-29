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

## Design Signal Handlers

In general, it is preferable to write simple signal handlers. One important reason for this is to
reduce the risk of creating race conditions. Two common designs for signal handlers are:

- The signal handler sets a global flag and exits. The main program periodically checks this flag
and, if it is set, takes appropriate action
- The signal handler performs some types of cleanup and then either terminates the process or uses a
nonlocal goto to unwind the stack and return control to a predetermined location in the main program

### Reentrant and Async-Signal-Safe Functions

A function is said to be _reentrant_ if it can safely be simultaneously executed by multiple threads
of execution in the same process. A function maybe _nonreentrant_ if it updates global or static
data structures. (A function that employs only local variables is guaranteed to be reentrant.)

Note that not all system calls and library functions can be safely called from a signal handler.

### Standard Async-Signal-Safe Functions

An _async\-signal\-safe_ function is one that the implementation guarantees to be safe when called
from a signal handler. A function is async-signal-safe either because it is reentrant or because it
is not interruptible by a signal handler

A function is unsafe only when invocation of a signal handler interrupts the execution of an unsafe
function, and the handler itself also calls an unsafe function. In other words, when writing signal
handlers, we have two choices:

- Ensure that the code of the signal handler itself is reentrant and that it calls only
async-signal-safe functions
- Block delivery of signals while executing code in the main program that calls unsafe functions or
works with global data structures also updated by the signal handler

### Terminating a Signal Handler

Two ways to terminate a signal handler:

- Normally 
  - Returning to the main program
- Abruptedly
  - Use _\_exit()_ to terminate the process. Beforehand, the handler may carry out some cleanup
  cleanup actions
  - Use _kill()_ or _raise()_ to send a signal that kills the process
  - Perform a nonlocal goto from the signal handler
  - Use the _abort()_ function to terminate the process with a core dump

### Handling a Signal on an Alternate Stack

Normally, when a signal handler is invoked, the kernel creates a frame for it on the process stack.
However, this may not be possible if a process attempts to extend the stack beyond the maximum
possible size.

Whena a process attemps to grow its stack beyond the maximum possible size, the kernel genereates a
_SIGSEGV_ signal for the process. However, since the stack space is exhausted, the kernel can't
create a frame for any _SIGSEGV_ handler that the program may have established. Consequently, the
handler is not invoked, and the process is terminated.

If we instead need to ensure that the _SIGSEGV_ signal is handled in these circumstances, we can do
the following:

1. Allocate an area of memory, called an _alternate signal stack_, to be used for the stack frame of
a signal handler
2. Use the _sigaltstack()_ system call to inform the kernel of the existence of the alternate signal
stack
3. When establishing the signal handler, specify the _SA\_ONSTACK_ flag, to tell the kernel that the
frame for this handler should be created on the alternate stack

```C
#include <signal.h>
int sigaltstack(const stack_t *sigstack, stack_t *old_sigstack);


typedef struct {
  void *ss_sp;       /* Starting address of alternate stack */
  int ss_flags;      /* Flags: SS_ONSTACK, SS_DISABLE */
  size_t ss_size;    /* Size of alternate stack */
} stack_t;
```

### Special Cases for Delivery, Disposition, and Handling

**_SIGKILL_** and **_SIGSTOP_**: It is not possible to change the default action for _SIGKILL_ and 
_SIGSTOP_, which always stops a process. These two signals also can't be blocked.

**_SIGCONT_** and **_stop signals_**: If a process is currently stopped, the arrival of a _SIGCONT_
signal always causes the process to resume, even if the process is currently blocking or ignoring
_SIGCONT_. This feature is necessary because it would otherwise be impossible to resume such stopped
processes. Whenever _SIGCONT_ is delivered to a process, any pending stop signals for the process
are discarded. Conversely, if any of the stop signals is delivered to a process, then any pending
_SIGCONT_ signal is automatically discarded. These steps are taken in order to prevent the action of
a _SIGCONT_ signal from being subsequently undone by a stop signal that was actually sent beforehand,
and vice versa.

### Interruptible and Uninterruptible Process Sleep States

At various time, the kernel may put a process to sleep, and two sleep states are distinguished:

- _TASK\_INTERRUPTIBLE_: The process is waiting for some event. A process may speedn an arbitrary
length of time in this state. If a signal is generated for a process in this state, then the
operation is interrupted and the process is woken up by the delivery of a signal (marked by the
letter _S_ in the _STAT_ field)
- _TASK\_UNINTERRUPTIBLE_: The process is waiting on certain special classes of event, such as the
completion of a disk I/O. If a signal is generated for a process in this state, then the signal is
not delivered until the process emerges from this state (marked with a _D_ in the _STAT_ field)
- _TASK\_KILLABLE_: This state is like _TASK\_UNINTERRUPTIBLE_, but wakes the process if a fatal
signal is received. By converting relevant parts of the kernel code to use this state, various
scenarios where a hung process requires a system restart can be avoided.

### Hardware-Generated Signals

_SIGBUS_, _SIGPFE_, _SIGILL_, and _SIGSEGV_ can be generated as a consequence of a hardware exception
or, less usually, by being sent by _kill()_. In the case of hardware exception, the behavior of a
process is undefined if it returns from a handler for the signal, or if it ignores or blocks the signal.
There is a number of reasons for it:

- _Returning from the signal handler_: causes an infinite loop
- _Ignoring the signal_: makes no sense since it is unclear how a program should continue
- _Blocking the signal_: makes no sense since  it is unclear how a program should continue

The correct way to deal with hardware-generated signals is either to accept their default action
(process termination) or to write handlers that don't perform a normal return. Other than returning
normally, a handler can complete execution by calling _\_exit()_ to terminate the process or by
calling _siglongjmp()_ to ensure that controll passes to some point in the program other than the
instruction that generated the signal

### Timing and Order of Signal Delivery

Synchronous signals are delivered immediately (self-generated signal or hardware-generated signal).
When a signal is generated asynchronously, there may be a (small) delay while the signal is pending
between the time when it was generated and the time it is actually delivered, even if we have not
blocked the signal. The reason for this is that the kernel delivers a pending signal to a process
only at the next switch from kernel mode to user mode while executing that process. That means the
signal is delivered at one of the following times:

- When the process is rescheduled after it earlier timed out
- At completion of a system call (delivery of the signal may cause a blocking system call to
completely prematurely)

If a process has multiple pending signals that are unblocked using _sigprocmask()_, then all of
signals are immediately delivered to the process. We shouldn't rely on the delivery order of multiple
signals. However, the standards provide guarantees about the order in which multiple unblocked
realtime signals are delivered in order

When multiple unblocked signals are awaiting delivery, if a switch between kernel mode and user mode
occurs during the execution of a signal handler, then the execution of that handler will be
interruped by the invocation of a second signal handler

![Delivery of multiple unblocked signals](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/delivery_multiple_unblocked_signals.drawio.png)

## Realtime Signals

Realtime signals have the following advantages over standard signals:

- Realtime signals provide an increased range of signals that can be used for application-defined
purposes. Only two standard signals are freely available for application-defined purposes: _SIGUSR1_
and _SIGUSR2_
- Realtime signals are queued. If multiple instances of a realtime signal are sent to a process,
then the signal is delivered multiple times. By contrast, if we send further instances of a standard
signal that is already pending for a process, that signal is delivered only once
- When sending a realtime signal, it is possible to specify data that accompanies the signal
- The order of delivery of different realtime signals is guaranteed. If multiple different realtime
signals are pending, then the lowest-numbered signal is delivered first or they are considered
higher priority

In order to send a realtime signal to a process, we need to use _sigqueue()_ system call
```c
#define _POSIX_C_SOURCE 199309
#include <signal.h>

int sigqueue(pid_t pid, int sig, const union sigval value);
```

### Handling Realtime Signals

Realtime signals can be handled just like standard signals, using a normal (sigle-argument) signal
handler. Alternatively, we can handle a realtime signal using a three-argument signal handler
established using the _SA\_SIGINFO_ flag

### Synchronously Waiting for a Signal

We can use the _sigwaitinfo()_ system call to synchronously _accept_ a signal. This approach can
help to avoid the need to write a signal handler and to handle the complexities of asynchronous
delivery of signals

```C
#define _POSIX_C_SOURCE 199309
#include <signal.h>

int sigwaitinfo(const sigset_t *set, siginfo_t *info);
```

_sigwaitinfo()_ system suspends execution of the process until one of the signals in the signal set
pointed to by _set_ is delivered. If one of the signals in _set_ is already pending at the time of
the call, _sigwaitinfo()_ returns immediately. The delivered signal is removed from the process's
list of pending signals, and the signal number is returned as the function result.

An alternative option is _sigtimedwait()_ system call, which allows us to specify a time limit
for waiting

```C
int sigtimedwait(const sigset_t *set, siginfo_t *info, const struct timespec *timeout);
```

Linux also provides _signalfd()_ system call, which creates a special file descriptor from which
signals directed to the caller can be read. The _signalfd_ mechanism provides an alternative to the
use of _sigwaitinfo()_ for synchronous accepting signals

```C
#include <sys/signalfd.h>

int signalfd(int fd, const sigset_t *mask, int flags);
```

Then we can use _read()_ to retrieve as many _signalfd\_siginfo_ structures as there are signals
pending and will fit in the supplied bufer. If no signals are pending at the time of the call, then
_read()_ blocks until a signal arrives. When no longer require a _signalfd_ fiel descriptor, it
should be closed in order to release the associated kernel resources

## Interprocess Communication with Signals

Signals can be considered as a form of interprocess communication. However, signals suffer a number
of limitations as an IPC mechanism.

- Asynchronous nature of signals means that we face various problems including reentrancy
requirements, race conditions, and the correct handling of global variables from signal handlers
- Standard signals are not queued. Event for realtime signals, there are upper limits on the number
of signals that may be queued. This means that in order to avoid loss of information, the process
receiving the signals must have a method of informing the send that it is ready to receive another
signal
- Signals carry only a limited amount of information
