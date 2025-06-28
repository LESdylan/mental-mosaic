# overview of learning 
|     |     |     |     |     |
| --- | --- | --- | --- | --- |
|     |     |     |     |     |
|     |     |     |     |     |
|     |     |     |     |     |
|     |     |     |     |     |
|     |     |     |     |     |
|     |     |     |     |     |
|     |     |     |     |     |

> we're building a system where a messenger (a user process) sends a personalized signal to a mini server, which is running in another terminal session. The system is distributed across mutliple terminals (sessions), but every process involved is uniquely identified by a [[pid]] That PID acts like a `unique beacon` or `road sign`, pointing us to the destination without needing to search for it.

## The messenger's journey 

![[Drawing 2025-06-20 14.43.16.excalidraw]]
### path 1
> hey are you the process ? are you ? are you ? are you ? until... are you ? -> yes, I am

This process is simply inneficient, the messegner will use a lot or ressources to get to the dest passing through a lot of pretendant dest like it , and anyway if he just know the whole path straightfoward, it would need to overcome several directories on its way.

### path2
> the messenger created a one way path to get direct access to the destination server, quicker and useful, but will ask for an extra chunk of memory and more system
> ressources(symlink tracking, file descriptor table).

the path is a direct access resulting a O(1)

### summary
so here we're trading with time for space -- classic time vs. space complexity problem. and over time, repeated O(1) access heats up the CPU significantly, since it needs more underying machinery to maintain the shortcut

Imagine a mailroom with thousands of mailboxes.
- we write a letter to roomm #348 -> (write a signal message)
- the mailroom dude sees "348" and knows exactly where to drop it -> (attach a PID to it)
- That 348 = routing key ->  (and our system knows which process to deliver it to)
# now how can we find this process in bash ?
[[process command]]
[[server]]
[[client]]

## process of transmission of information
The transmission

![[Drawing 2025-06-20 17.49.47.excalidraw]]

> Acknoledgement system:
> for each bit sent, 
> the client sends signal and wait for acknowledgement
> server send SIGUSR1 back to client after receiving each bit

```bash
1. Send 'h' (8 signals) → Server acknowledges → Adds 'h' to buffer
2. Send 'e' (8 signals) → Server acknowledges → Adds 'e' to buffer  
3. Send 'l' (8 signals) → Server acknowledges → Adds 'l' to buffer
4. Send 'l' (8 signals) → Server acknowledges → Adds 'l' to buffer
5. Send 'o' (8 signals) → Server acknowledges → Adds 'o' to buffer
6. Send '\0' (8 signals) → Server sends SIGUSR2 → Prints "hello\n"

```
total 48 signals just 6 bytes, plus each signal requires acknowledgment, so thre are actually 96 total signal exchanges (648 data+ 48 acks)

# Optimization strategies
### packed transmission (most efficient)
instead of sending one character at a time, pack multiple characters into a single transmission:
### Variable-length encoding
use fewer bits for common characters
- ASCII 32-127(printable) = 7 bits instead of 8
- common letter could use even fewer bits with [[Huffman]]
### block acknowledgements
instead of acknowledging every bit, acknowledge every character or every N characters

### protocol optimization
- remove null terminator : send length prefix instead
- batch mode : send mutliples messages before requiring acks
- compression: use simple compression for repeated characters

## Huffman compression
![[huffman algo]]

# batch bit
batch bit for faster transmission while keeping huffman protocol and checksum logic
1. pack bits into bytes on teh client side before sending
2. send each byte using 8 signals 
3. unpack bits on the server side to reconstruct the bitstream for huffman decoding and checksum calculation

```c
// Packs a string of '0'/'1' bits into bytes, returns number of bytes
int pack_bits(const char *bitstr, unsigned char *out) {
    int len = strlen(bitstr);
    int byte_count = (len + 7) / 8;
    for (int i = 0; i < byte_count; i++) {
        out[i] = 0;
        for (int b = 0; b < 8; b++) {
            int idx = i * 8 + b;
            if (idx < len && bitstr[idx] == '1')
                out[i] |= (1 << (7 - b));
        }
    }
    return byte_count;
}
```


```c
unsigned char packed[1250]; // 10000 bits max / 8 + some margin

int packed_len = pack_bits(ctx.encoded, packed);

for (i = 0; i < packed_len; i++) {

for (int b = 7; b >= 0; b--) {

char bit_char = ((packed[i] >> b) & 1) ? '1' : '0';

send_bit(server_pid, bit_char);

}

}
```

Thank you for clarifying! If you are limited to standard UNIX signals (SIGUSR1/SIGUSR2), you cannot send a full byte in a single signal—only a single bit per signal is possible. This is a fundamental limitation of the UNIX signal API: each signal can only represent one of two states (SIGUSR1 or SIGUSR2), not a value from 0–255.

**Summary:**

- With only SIGUSR1/SIGUSR2, you can only send one bit per signal.
- Packing bits into bytes is only useful for memory efficiency, not for transmission speed, unless you can use signals with payloads (which you cannot).

**What you have now is optimal for your constraints:**

- You are sending one bit per signal, as fast as possible.
- The only way to go faster is to reduce the delay (`usleep`) or use a different IPC mechanism (like sockets or shared memory), which is outside the scope of UNIX signals.

**If you want to further optimize:**

- Tune the `usleep` value for your system.
- Optionally, implement an ACK mechanism so the client only sends the next bit after the server processes the previous one (to avoid lost signals at very high speed).

If you need help with ACK-based flow control or further tuning, let me know!

- 
- 
- 

3 files changed

KeepUndo

huff_encode.ccipher-vault/huffman

minitalk_client.ccipher-vault/huffman

minitalk_server.ccipher-vault/huffman

- Add Context...

minitalk_client.c:5-24

Current file