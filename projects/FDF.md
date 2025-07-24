---
banner: Pasted image 20250724133817.png
banner_y: "31"
---
# PARSER

## 2 pass buffer
## chunked buffer size
Chunked parser  reads data in fixed-size chunks (buffers), processes each chunk, and  handles leftover  data between reads. this is a standard and efficient way to parse large files or streams without loading the entire  content  into memory.

## Difference between get_next_line and chunked_algo
Chunked parser:
- Reads large, fixed-size chunks (4096 bytes) from the file.
- Processes as much data as possible from each chunk, handling leftover between reads
- efficient for parsing large files or streams, as it minimies system calls and memory alllocatons.
- Can process multiple lines or tokens per  read, not  limited  to line  boundaries.
- Good for  custom parsing logic (parsing by delimiter, not just newlines)
Get_next_line:
- Reads and returns one line at a time from a file descriptor.
- typically allocates memory for each lineand may call `read` multiple  times to find the end of  a line
- Simpler to use when we only need  to process files liny by line
- less efficient for very large files, as it may involve more system calls and allocations.
- Not suitable for parsing  data that isn't line-based o when we want to process in  larger  blocks
> chunk parser is more efficient for large files, custom parsing and non-line-based data.

# Framework of tracking exceptions in C

T