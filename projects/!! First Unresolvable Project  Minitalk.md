---
aliases:
  - "!! First Unresolvable Project : Minitalk"
---
[SIGNAL]
https://en.wikipedia.org/wiki/Volatile_(computer_programming)

---
Just when we start to get used to give our maximum to get our program errors free with 42 school project. I discovered throughout a lot of trial and failure that this problem is impossible because of the requirements of the school.
The core problem I primarily identified in this project is this kind of pattern
```c
//this sequence ALWAYS has a race condition:
volatile sig_atomic_t signal_received = 0;

void handler(int sig) {
    signal_received = 1;
    printf("Signal received at time T2\n");
}

// Main code:
signal(SIGUSR1, handler);

if (!signal_received) {              // T1: Check (returns false)
    // Server sends SIGUSR1 HERE     // T2: Signal arrives, handler runs
    pause();                         // T3: pause() waits for NEXT signal
}

## // my solution
while (!server->ready_to_proceed) { usleep(100); // Even if signal is "lost", we check again in 100μs }

//if signal arrive between step 1 and step2, pause hangs() forever
// since pause() only wakeds up on new signals that arrive after it starts, and our signal can arrive before, it will wait forever for another signal

//and as we ask the signal to be received the condition can never be accomplished
```

Why our requirements are problematic ?
The restriction to only use `signal()`/ `sigaction` without proper signal `masking primitives`makes reliable singla hanlding impossible. This is exactly why POSIX introduced `sigprocmask()` , `sigusped()` and `signalfd()`
As all project we can do in our life, a difficulty of the project can reflect the mind spirit of the developer that codes it. This project is one of them because a we see too much from my grade for example that people do the easiest choice possible 

## Concepts to see in the nutshell
### identifying each role between server and client
#### Client
#### Server
### Signals
unix give a safe 
Primarily the signal was created to 

> Portability
       The only portable use of signal() is to set a signal's disposition to SIG_DFL or SIG_IGN.  The semantics when using signal() to establish a signal handler vary across systems (and POSIX.1 explicitly
       permits this variation); do not use it for this purpose.
       POSIX.1 solved the portability mess by specifying sigaction(2), which provides explicit control of the semantics when a signal handler is invoked; use that interface instead of signal().
       In  the original UNIX systems, when a handler that was established using signal() was invoked by the delivery of a signal, the disposition of the signal would be reset to SIG_DFL, and the system did
       not block delivery of further instances of the signal.  This is equivalent to calling sigaction(2) with the following flags:
           sa.sa_flags = SA_RESETHAND | SA_NODEFER;
       System V also provides these semantics for signal().  This was bad because the signal might be delivered again before the handler had a chance to reestablish itself.  Furthermore,  rapid  deliveries
       of the same signal could result in recursive invocations of the handler.
       BSD  improved  on this situation, but unfortunately also changed the semantics of the existing signal() interface while doing so.  On BSD, when a signal handler is invoked, the signal disposition is
       not reset, and further instances of the signal are blocked from being delivered while the handler is executing.  Furthermore, certain blocking system calls are automatically restarted if interrupted
       by a signal handler (see signal(7)).  The BSD semantics are equivalent to calling sigaction(2) with the following flags:
           sa.sa_flags = SA_RESTART;
       The situation on Linux is as follows:
       * The kernel's signal() system call provides System V semantics.
 
| command | summary                                                                                                                                                                | page         |                                                                                                                       |                                                                                          |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| kill    |                                                                                                                                                                        | man 7 signal |                                                                                                                       |                                                                                          |
| pause   | cause the calling process (`or thread`) t sleep until a signal is delivered that either terminates the process or causes teh invocation of a signal-catching function. | man 2 pause  | return only when signal was caught and the signal-catching funtion returned  otherwise (-1) and errno is set to EINTR | may be handled into the handler_funciton to be seu that the signal is received each time |
| signal  |                                                                                                                                                                        | man 7 signal |                                                                                                                       |                                                                                          |
#### SIGSET 
#### SIGACTION
#### SIGNAL
Let's imagine we're writing to a file, updating a pointer, or halfway through some malloc.
A signal handler can suddenly JUMP IN like:
`I'M SIGUSR1`
but the program lost why it needed this

signal handler, we cannot malloc, we can't use mutexes, we can't print safely
we can only set a volative sig_atomic_t flag and run
### ICP

### Limits of signals

the signal rely on `async interruptions`
we can have `race condition` issues, `override values` because `not queued` 
insie

---
## inline Vocabulary
### volatile
`volatile sig_atomic_t`  : volatile is a keywork that modifies a variable indicating that its value may change at any time by external factors (like hardware or other threads), and therefore the compiler should not make assumptions about its value. Writing this `indicator`make the compiler reading directly into the `memory` rather than reling on `cached values` this ensures that all threas onprocesses see the most up-to-date value. The memory is the most update value from the system can get because the cache `can't notice teh environment/headers change, compiler flags change, when timestamp are messy, randomness or harware..` --> this explain why we need to use volatile
volatile is a `type qualifier` like `const` 
- `volatile` access to `memory-mapped` I/O devices.
- allow preserving value across a `longjump` 
- allow sharing values between signal handlers and the rest of the program in `volatile`

