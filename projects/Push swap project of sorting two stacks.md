>objective : The objective is to sort Stack A in ascending order (smallest element at the top) using the fewest operation possible, with Stack B empty at the end. 

Push Swap is a classic algorithm project from the 42 School curriculum, designed to deepen your understanding of sorting algorithms, data structures, and optimization techniques. The goal is to sort a stack of integers using a limited set of operations and two stacks, Stack A and Stack B, with the fewest moves possible. This project challenges you to think critically about algorithm efficiency, not in terms of computational time but in terms of minimizing the number of operations.

---
## Understanding the Push Swap Problem
The project restricts you to a specific set of operations to manipulate the stacks:
![[test.webp]]
- **sa** (swap a): Swap the first two elements at the top of Stack A.
- **sb** (swap b): Swap the first two elements at the top of Stack B.
- **ss**: Perform sa and sb simultaneously.
- **pa** (push a): Take the top element from Stack B and push it to the top of Stack A. Do nothing if Stack B is empty.
- **pb** (push b): Take the top element from Stack A and push it to the top of Stack B. Do nothing if Stack A is empty.
- **ra** (rotate a): Shift all elements of Stack A up by one, with the top element becoming the last.
- **rb** (rotate b): Shift all elements of Stack B up by one, with the top element becoming the last.
- **rr**: Perform ra and rb simultaneously.
- **rra** (reverse rotate a): Shift all elements of Stack A down by one, with the last element becoming the first.
- **rrb** (reverse rotate b): Shift all elements of Stack B down by one, with the last element becoming the first.
- **rrr**: Perform rra and rrb simultaneously.

## constraints
- The input is  a space separated list of unique integers (e.g., `./push_swap 3 1 2 4`)
- The program must handle any valid input, including negative numbers and duplicate are not allowed
- The goal is to minimize the number of operations. For evaluation at 42 school, specific threshold exist : 
	- For 3 numbers : <= 3 operations
	- for 5 numbers: <= 12 operations
	- for 100 nnumbers <= 700 operations for the highest score
	- for 500 numbers: <= 5,500 operations for the highest score
## Checker program
Alongside `push_swap` , we typically write a `checker` program that verifies if the output operations correctly sort the stack. The checker takes the same input integers and reads the operations from standard input, then outputs "OK" if the stack is sorted and "KO" otherwise.

## Designing the Push Swap Algorithm
### Data structure
For efficiency, I recommend to implement either a **circular buffer** (an array-based structure) rather than a linked list. A circular buffer allows faster access and manipulation of elements, which is critcical when debugging and testing with large inputs. The stack structure should include:
- An array to store integers
- A size variable to track the number of elements
- A top index to indicate the top of stack.

For example in C:
```c
typedef struct s_stack
{
	int *data;
	int size;
	int top;
}
```
This structure is efficient for operations like  **push, pop, rotate, swap** as it avoids the overhead of dynamic memory allocation in  `linked lists`. 
> [!NOTE]
> don't do like me, i used different structure for different algorithms, creating redundancy, spaguetty code and increase confusion into lecture. If I had more time I would clean all of those confusions...

## Algorithm strategies
## Permutations

### Chunk sort
#### Vision 
The algorithm's vision is to **divide and conquer** by breaking the sorting problem into manageable chunks, distributing them accros two stacks and recursively sorting smaller sub-chunks. Key aspect of the vision include:
1. **Chunk-based Partitionning** 
	- Instead of sorting the entire stack at once, the algorithm splits it into smaller chunks based on value ranges.
	- This reduces complexity by working with smaller, more manageable subsets of numbers.
2. Two Stack utilization:
	- Stack A is used to build the sorted seqeunce (typically in ascending order from bottom to top)
	- Stack B holds temporary chunks or partially sorted sub-chunks, allowing parallel processing of different value ranges.
3. Pivot-based splitting:
	- By using two pivots, the algorithm divides a chunk into three sub-chunks (`min` , `mid`, `max`), which balances the workload across recursive calls
	- The pivots are chosen to create roughly equal-sized sub-chunks, optimizing the number of operations.
4. Recursive Sorting:
	- Each sub-chunk is recursively split until its size is `<= 3` , at which point simple sorting functions (`sort_three`, `sort_two` , `sort_one`) are used.
	- The recursion ensures that smaller chunks are fully sorted before merging back into the main stack
5. optimization:
	- `the chunk_to_the_top` function ensures are in optimial positions reducing unnecessary rotations. 
6. Scalability:
	- the algorithm is designed to handle large inputs efficiently by keeping the number of operatoions low through strategic chunking and merging...

Divide the input into chunks (e.g., groups of 20-50 numbers based on their value ranges). Push each chunk to Stack B in a way that maintains partial order, then merge back to Stack A optimally.. This requires careful cost analysis to minimize rotations and pushes.
#### How it splits stack into sub-chunks
The algorithm splits a chunk into three sub-chunks (`min`, `mid`, `max`) and distributes them across Stack A and stack B. Here's how it effectively "cuts the two stacks into four stacks" (though it's more about logical partitioning within the two physical stacks):
#### initial  Setup
- start with a chunk in Stack (e.g., `to->sort->loc = TOP_A`, size = `n`)
- Stack b is typically empty or contains previously processed chunks.

#### Split Process
- **pivots**: calculate `pivot 1` (2/3 of size) and `pivot 2` (1/3 of size) based on the chunk size
- **value ranges**:
	- `max` : values in the top third (largest values, `> max_value - pivot 2`)
	- `mid` : values int the middle third (`max_value - pivot 1 < value <= max_value - pivot2)`
	- `min` : Values in the bottom third (smallest values, `<= max_value - pivot 1` )
- **Movement**:
	- Elements are moved using `move_from_to`, which may involve `pb` (push to B), `pa` (push to A), `ra`, `rb` etc...
	-  The `set_split_loc` function determines where each sub-chunk goes:
		- For `TOP_A` : `max` -> `BOTTOM_A` , mid, ->`TOP_B`, `min` -> `BOTTOM_b`
		- This creates logical partitions: `BOTTOM_A` and `TOP_A` in Stack A , `TOP_B` and `BOTTOM_B` in stack B
- **Result** the original chunk is split into three sub-chunks, distributed across the two stacks, effectively creating four logical `stacks` (two regions in Stack A, two in Stack B)
#### Recursive Processing
Each sub-chunk (`max`, `mid` ,`min`) is processed recursively.
1. max is sorted firs, often staying in Stack A to build the sorted sequence
2. `mid` and `min` are processed next, typically in Stack B, and eventually moved back to Stack A
3. The recursion continues until all chunks are sorted.

### Greedy sort
The algorithm is called greedy because it makes locally optimal choices at each step (e.g., choosing the element with the lowest cost to move) with the goall of achieving a globally eficient solution. It contrasts with the previously discussed `rec_chunk_sort`, which uses a recursive divide-and-conquer approach with chunk splitting. The greedy approach is simpler and often more intuitive, though it may not always produce the absolute minimum number of operation compared to chunk-based sort

- `greedy_sort` : The main entry point, orchestrating the three phases of the algorithm.
- `greedy_push_to_b` : moves element form stack A to Stack B, reducing stack A to 3 elemments and organizing Stack B
- `greedy_push_back_to_a`: Moves elements back from Stack B to Stack A, choosing the cheapest move each time.
- `greedy_final_rotation` : Rotates Stack A to place the smallest element at the top.
- Helper Functions:
	- `greedy_push_element_strategically` : Decides how to push elements to stack B on their value
	- `greedy_calculate_costs`: Computes the cost of moving each element from Stack B to its target position in Stack A.
- `greedy_find_cheapest`: Identifies the element in Stack B with the lowest movement cost.
- `greedy_execute_move`: Performs the rotations and push to move an element from Stack B to Stack A.
- `greedy_find_target_position`: Finds the optimal position in Stack A for an element from Stack B.
- `greedy_rotate_to_top`: Rotates Stack A to position the smallest element at the top.

```c
void greedy_sort(t_ps *data)
{
    if (!data || data->total_size <= 3)
    {
        if (data->total_size == 3)
            sort_three_simple(data);
        else if (data->total_size == 2
            && get_stack_element_at_position(&data->a, 1)
            > get_stack_element_at_position(&data->a, 2))
            sa(data);
        return;
    }
    if (verify_stack_is_sorted(data))
        return;
    data->algo_ctx.ctx.greedy.phase = 0;
    greedy_push_to_b(data);
    data->algo_ctx.ctx.greedy.phase = 1;
    greedy_push_back_to_a(data);
    greedy_final_rotation(data);
}

```
This is the entry point for the greedy algorithm,handling the entire sorting process.
**Steps**:

1. **Base Cases**:
    - If the input size is ≤ 3, sort directly:
        - For size = 3, call sort_three_simple to sort Stack A.
        - For size = 2, swap (sa) if the top two elements are out of order.
        - For size ≤ 1 or if Stack A is already sorted (verify_stack_is_sorted), return immediately.
2. **Phase 0: Push to Stack B**:
    - Set phase = 0 in the algorithm context and call greedy_push_to_b to move most elements to Stack B, leaving 3 elements in Stack A.
3. **Phase 1: Push Back to Stack A**:
    - Set phase = 1 and call greedy_push_back_to_a to move elements back to Stack A in the correct order.
4. **Final Rotation**:
    - Call greedy_final_rotation to ensure the smallest element is at the top of Stack A.

### Tag sort

### Radix sort
This is a popular choice for Push Swap because it's non-comparative and can be adapted to work with stacks, bur far from be good enough to get the highest score. 
Map the integers  to their ranks (e.g., 1024, 19, 42, -5119, becomes 4, 2, 3, 1) to simplify comparisons. Then, sort based on binary digits (LSB to MSB), pushing numbers to Stack B based on the current bit and rotating as needed This approach is efficient and can sort 500 operations with `6,500` movements.

## Optimizations strategies
### Longest increasing subsequences (LIS)
idetify the `LIS` in Stack A and leave those element in place, pushing others to Stack B This reduces the number of operations by keeping already sorted elements untouched.

### Operation merging:
Combine operations like `ra` and `rb` into `rr` when possible to reduce the total count

> [!NOTE]
  more the numbers movement variation is wide more we need those optimizations
  For example in radix_sort the variation is really little we don't have too much merge to do optimizatons..
### Rank Mapping
Debugging push swap is challenging because the output is  a sequence of operations, and it's hard to visualize the state of stacks.. This is where a visualizer becomes invaluable. Many exisiting visualizers are graphical (Godot...), but I've 
developed my version of algorithm


## Terminal-Based WebP Visualizer for Debugging
Graphical visuallizers,, like those built with SFML, Godot, or web technologies, are powerful but have drawbacks:
- Setup complexity: They require installing dependencies, which can be cumbersome, especially on systems like 42
- Ressource Usage: Graphical interfaces consume more system ressources, slowing browser limiting accessibility.
- Portability: They often require specific environments or a browser, limiting accessibility.
- Learning curve : Undestanding and navigating a GUI can distract from debugging algortihm itself

To address these issues, i created a  `terminal-based visualizer` that generates a tow stacks and the movements of your algorithm highly effective for debugging.  

The visualizer is a program  that : 
- takes the `push_swap` executable and list of integers as input.
- render each stateas a simple ASCII-art representation in the terminal.


As you can see in this video 