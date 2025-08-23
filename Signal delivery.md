Signals are delivered by the **kernel** as asynchronous notifications to processes. Each signal is represented by an integer identifier, and when an event occurs (for example, the user pressing `Ctrl+C`), the kernel decides which process or group of processes should receive that signal. Delivery interrupts the normal execution flow of the target process, forcing it to respond immediately.

The **process** that receives the signal can react in one of three ways:

- Use the **default action** associated with the signal (terminate, stop, ignore, or dump core).
    
- **Ignore** the signal, if it is allowed (some signals, like `SIGKILL`, cannot be ignored).
    
- Install a **custom handler**—a function defined by the program that runs automatically when the signal arrives. This allows applications such as shells to override the default behavior and provide a user-friendly response.


## Examples
### 1. Default action (terminate with `SIGINT`)  
```c
#include <signal.h>
#include <unistd.h>

int main(void) {
    struct sigaction sa;
    sa.sa_handler = SIG_DFL;    // usar acción por defecto
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    sigaction(SIGINT, &sa, NULL);

    while (1) pause();          // Ctrl+C → programa termina
}

    ```
### 2. Ignore signal (`SIGINT`)  
```c
#include <signal.h>
#include <unistd.h>

int main(void) {
    struct sigaction sa;
    sa.sa_handler = SIG_IGN;    // ignorar SIGINT
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    sigaction(SIGINT, &sa, NULL);

    while (1) pause();          // Ctrl+C → no hace nada
}
```

### 3. Custom handler (`SIGINT`)  
```c
#include <signal.h>
#include <unistd.h>

void handler(int sig) {
    (void)sig;
    write(STDOUT_FILENO, "Caught SIGINT\n", 14);
}

int main(void) {
    struct sigaction sa;
    sa.sa_handler = handler;    // manejador propio
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    sigaction(SIGINT, &sa, NULL);

    while (1) pause(); // Ctrl+C → imprime mensaje
}
```



