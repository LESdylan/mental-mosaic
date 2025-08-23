Signals are a core component of process management in Unix-like systems, enabling asynchronous communication between the kernel and user-space programs. They inform processes of events such as interrupts, termination requests, or exceptions, and dictate how those processes should react. In the context of Minishell, signal handling is critical to reproducing the behavior of a real shell. Implementing correct responses to signals like `SIGINT` (`Ctrl+C`) or `SIGQUIT` (`Ctrl+\`) ensures consistency with standard shells and maintains stable, predictable interactions for the user.

Here are the most common signals for Minishell:

- **[[SIGINT]] (2)** → Sent when the user presses `Ctrl+C`. Default action: terminate the process.
    
- **[[SIGQUIT]] (3)** → Sent with `Ctrl+\`. Default action: terminate the process and dump core.
    
- **[[SIGCHLD]] (17)`** → Sent to a parent when a child process terminates or stops.


👉 For **Minishell**, the most relevant are **`SIGINT`**, **`SIGQUIT`**, and sometimes **`SIGCHLD`**.
### Signal behavior: Bash vs. Minishell

| **Signal**                 | **In Bash (parent shell)**                                                   | **In Bash (child process)**                      | **In Minishell (what to do)**                                                                        |
| -------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| **[[SIGINT]] (`Ctrl+C`)**  | Clears current input line, shows new prompt, shell does **not** exit.        | Terminates the running program.                  | Parent: install custom handler (refresh prompt).  <br>Child: restore default, let program terminate. |
| **[[SIGQUIT]] (`Ctrl+\`)** | Ignored, shell stays alive.                                                  | Terminates program, prints `Quit (core dumped)`. | Parent: ignore `SIGQUIT`.  <br>Child: restore default, allow program to terminate with core dump.    |
| **[[SIGCHLD]]**            | Not visible to user, but Bash reaps children to avoid zombies and sets `$?`. | Sent to parent when child ends.                  | Parent: use `waitpid()` to collect exit status, update `$?`, prevent zombies.                        |
