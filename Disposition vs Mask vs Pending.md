### Disposition vs Mask vs Pending — the trio you must distinguish

- **Disposition (what to do when it arrives)**  
    Set per-signal with **`sigaction`** to one of: `SIG_DFL` (default), `SIG_IGN` (ignore), or a custom handler.  
    Immutable exceptions: **`SIGKILL`** and **`SIGSTOP`** (can’t catch/ignore/change).
    
- **Mask (which signals are temporarily blocked)**  
    A **per-thread** bitmap telling the kernel “don’t deliver these now.”  
    Manage with **`sigprocmask`** (or **`pthread_sigmask`** in threaded code).  
    While masked, matching signals become _pending_ instead of being delivered.
    
- **Pending set (which signals are waiting)**  
    At most **one pending instance per signal number**.  
    Inspect with **`sigpending`**. They’re delivered when you later unblock them.


```c
#include <signal.h>
#include <unistd.h>
#include <stdio.h>

int main(void){
    sigset_t set, old, pend;
    sigemptyset(&set); sigaddset(&set, SIGINT);

    pthread_sigmask(SIG_BLOCK, &set, &old);     // 1) block SIGINT (per-thread)
    puts("SIGINT blocked. Press Ctrl+C now..."); sleep(3);

    sigpending(&pend);                          // 2) check pending
    if (sigismember(&pend, SIGINT)) puts("SIGINT is pending");

    pthread_sigmask(SIG_SETMASK, &old, NULL);   // 3) unblock -> now delivered
    pause();                                    // wait for handler/default
}
```

