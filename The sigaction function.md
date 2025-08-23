## 🌐 Purpose of `sigaction`

- `sigaction` is the **POSIX function** to tell the kernel _how your process should react when it receives a given signal_.
    
- Without it, signals just do their **default action** (terminate, stop, ignore, core dump).
    
- With it, you can:
    
    - Replace the default with your **own handler function**.
        
    - Explicitly set it back to **ignore** or **default**.
        
    - Control advanced behavior (block masks, restart syscalls, extra info with `sa_sigaction`).
        

---

## 📝 Prototype

`int sigaction(int signum, const struct sigaction *act,               struct sigaction *oldact);`

- **`signum`** → the signal number (e.g., `SIGINT`, `SIGQUIT`).
- **`act`** → pointer to a `struct sigaction` describing the _new action_ for this signal.
- **`oldact`** → if not `NULL`, the kernel fills it with the _previous action_ (so you can restore later).

Return value: `0` on success, `-1` on error (with `errno` set).

---

## 📦 The `struct sigaction`

struct sigaction
{     
	void     (*sa_handler)(int);                                   // basic handler function     
	void     (*sa_sigaction)(int, siginfo_t *, void *); // advanced handler
	sigset_t sa_mask;                                             // signals to block during handler
	int      sa_flags;                                                 // options controlling behavior
	void     (*sa_restorer)(void);                             // obsolete (ignore in practice)
};

### Key fields you’ll actually use:

- **`sa_handler`**
    - Pointer to your handler function: `void handler(int sig)`
    - Or special values:
        - `SIG_DFL` → restore default action.
        - `SIG_IGN` → ignore signal.
- **`sa_sigaction`**
    - Alternative handler with more info: `void handler(int, siginfo_t*, void*)`.
    - Activated if you set `sa_flags |= SA_SIGINFO`.
- **`sa_mask`**
    - Set of signals that will be _blocked automatically_ while the handler runs.
    - Usually initialized with `sigemptyset(&sa.sa_mask)`.
- **`sa_flags`**
    - Extra options. Common ones:
        - `SA_RESTART` → restart syscalls interrupted by signals.
        - `SA_SIGINFO` → use `sa_sigaction` instead of `sa_handler`.