# Virtual Memory Manager

A comprehensive virtual memory management simulator that implements multiple page replacement algorithms for operating systems. This project demonstrates how different page replacement policies handle page faults and manage memory frames efficiently.

## Overview

This virtual memory manager takes a sequence of page references as input along with the number of available frames, and performs page placement using the page replacement policy specified by the user. The system compares the performance of the selected algorithm against the Optimal page replacement algorithm, providing detailed metrics including page fault counts, execution time, and performance penalties.

## Features

### 1. Multiple Page Replacement Algorithms

The simulator implements five different page replacement algorithms, each with unique characteristics:

#### **FIFO (First-In-First-Out)**
- **Description**: The simplest page replacement algorithm that replaces the oldest page in memory.
- **Implementation**: Uses a singly linked list to maintain pages in the order they were loaded.
- **Behavior**: When a page fault occurs and all frames are full, the page that has been in memory the longest is replaced.
- **Use Case**: Easy to implement but may not always provide optimal performance.
- **Time Complexity**: O(1) for page replacement, O(n) for duplicate checking where n is the number of frames.

#### **LFU (Least-Frequently-Used)**
- **Description**: Replaces the page that has been referenced the least number of times.
- **Implementation**: Uses a doubly linked list and counts the frequency of each page reference in the history.
- **Behavior**: When a page fault occurs, the algorithm counts how many times each page in memory has been referenced up to the current point, and replaces the one with the lowest count.
- **Use Case**: Effective for workloads where page access patterns show clear frequency differences.
- **Time Complexity**: O(n × m) where n is frames and m is the current position in the reference string.

#### **LRU-STACK (Least-Recently-Used - Stack Implementation)**
- **Description**: Replaces the page that has not been used for the longest time, using a stack-based approach.
- **Implementation**: Uses a doubly linked list where recently accessed pages are moved to the end (most recently used), and the least recently used page is at the head.
- **Behavior**: 
  - When a page is accessed, it's moved to the end of the list (most recently used position).
  - When a page fault occurs, the page at the head (least recently used) is replaced.
- **Use Case**: Excellent for temporal locality in page access patterns.
- **Time Complexity**: O(n) for finding and moving pages, where n is the number of frames.

#### **LRU-CLOCK (Least-Recently-Used - Clock/Second Chance Algorithm)**
- **Description**: An approximation of LRU using a circular buffer with reference bits (second chance algorithm).
- **Implementation**: Uses a circular linked list where each node has a reference bit.
- **Behavior**:
  - Each page has a reference bit that is set to 1 when the page is accessed.
  - When a page fault occurs, the algorithm scans the circular list:
    - If a page's reference bit is 0, it's replaced immediately.
    - If a page's reference bit is 1, it's set to 0 and the algorithm continues to the next page (giving it a "second chance").
  - This continues until a page with reference bit 0 is found.
- **Use Case**: More efficient than LRU-STACK for large numbers of frames, as it avoids moving pages in the list.
- **Time Complexity**: O(n) worst case for finding a page to replace, but typically better in practice.

#### **LRU-REF8 (Least-Recently-Used - 8 Reference Bits)**
- **Description**: An enhanced LRU algorithm that uses 8 reference bits to track page usage history.
- **Implementation**: Uses a doubly linked list and maintains an 8-bit reference history for each page.
- **Behavior**:
  - For each page in memory, the algorithm looks back at the last 8 page references.
  - Creates an 8-bit binary pattern where 1 indicates the page was referenced and 0 indicates it wasn't.
  - Converts this binary pattern to a decimal value.
  - Replaces the page with the lowest decimal value (least recently referenced in the last 8 accesses).
- **Use Case**: Provides a balance between accuracy and efficiency, using recent history to predict future needs.
- **Time Complexity**: O(n × 8) where n is the number of frames.

#### **Optimal Algorithm (For Comparison)**
- **Description**: The theoretically optimal page replacement algorithm that replaces the page that will be used farthest in the future.
- **Implementation**: Uses a doubly linked list and looks ahead in the reference string to determine which page won't be needed soonest.
- **Behavior**: When a page fault occurs, the algorithm checks when each page in memory will be referenced next, and replaces the one that will be referenced farthest in the future (or never).
- **Use Case**: Used as a benchmark for comparing other algorithms. Not practical in real systems as it requires knowledge of future page references.
- **Time Complexity**: O(n × m) where n is frames and m is the remaining reference string length.

### 2. Command Line Interface

The program provides a flexible command-line interface with the following options:

```
virtualmem [-h] [-f available-frames] [-r replacement-policy] [-i input_file]
```

#### Command Line Options:

- **`-h`**: 
  - Prints a detailed usage summary with all available options and exits.
  - Displays help information for users unfamiliar with the program.

- **`-f available-frames`**: 
  - Sets the number of available frames in physical memory.
  - **Default**: 5 frames
  - **Example**: `-f 10` sets the number of frames to 10.
  - **Validation**: Must be a positive integer.

- **`-r replacement-policy`**: 
  - Sets the page replacement policy to use.
  - **Default**: FIFO
  - **Valid Options**:
    - `FIFO` - First-In-First-Out
    - `LFU` - Least-Frequently-Used
    - `LRU-STACK` - Least-Recently-Used (Stack Implementation)
    - `LRU-CLOCK` - Least-Recently-Used (Clock/Second Chance)
    - `LRU-REF8` - Least-Recently-Used (8 Reference Bits)
  - **Case Insensitive**: The policy name is automatically converted to uppercase.
  - **Example**: `-r LFU` or `-r lfu` both work.

- **`-i input_file`**: 
  - Specifies the input file containing the page reference sequence.
  - **Format**: The file should contain space, tab, or newline-separated page numbers.
  - **Example**: `-i input.txt`
  - **If not specified**: The program reads the page reference sequence from standard input (STDIN).
  - **Validation**: All input values must be numeric (digits only).

### 3. Input Processing

The system provides robust input handling with the following features:

- **File Input**: Reads page reference sequences from a specified file.
- **Standard Input**: Falls back to reading from STDIN if no file is provided.
- **Tokenization**: Automatically splits input by spaces, tabs, and newlines.
- **Validation**: 
  - Ensures all page references are numeric (digits only).
  - Validates frame count is numeric.
  - Provides clear error messages for invalid input.
- **Flexible Format**: Accepts page references separated by any whitespace character.

**Example Input Format:**
```
7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1
```

### 4. Performance Metrics and Output

The program provides comprehensive performance analysis comparing the selected algorithm against the Optimal algorithm:

#### Output Metrics:

1. **Page Replacement Count**:
   - Shows the number of page faults (page replacements) for the selected algorithm.
   - Shows the number of page faults for the Optimal algorithm.
   - Format: `# of page replacement with [ALGORITHM] : [COUNT]`

2. **Performance Penalty**:
   - Calculates the percentage penalty of the selected algorithm compared to Optimal.
   - Formula: `((Selected - Optimal) / Optimal) × 100`
   - Indicates how much worse the selected algorithm performs compared to the theoretical best.
   - Format: `% page replacement penalty using [ALGORITHM] : [PERCENTAGE]%`

3. **Execution Time**:
   - Measures the execution time in microseconds for both algorithms.
   - Uses `gettimeofday()` system call for precise timing.
   - Format: `Total time to run [ALGORITHM] algorithm : [TIME] msec`

4. **Speed Comparison**:
   - Compares the execution speed of the selected algorithm against Optimal.
   - Shows whether the selected algorithm is faster or slower.
   - Calculates the percentage difference in execution time.
   - Format: `[ALGORITHM] is [PERCENTAGE]% faster/slower than Optimal Algorithm`

#### Example Output:
```shell
$ virtualmem –f 10 –r LFU –i myinputfile
# of page replacement with LFU : 118
# of page Replacement with Optimal : 85
% page replacement penalty using LFU : 38.8%

Total time to run LFU algorithm : 1214 msec
Total time to run Optimal algorithm : 1348 msec
LFU is 9.9% faster than Optimal algorithm.
```

### 5. Data Structures

The implementation uses efficient data structures tailored to each algorithm:

- **Singly Linked List (FIFO)**: Simple linear structure for maintaining insertion order.
- **Doubly Linked List (LRU-STACK, LFU, LRU-REF8, Optimal)**: Allows efficient insertion, deletion, and movement of nodes in both directions.
- **Circular Linked List (LRU-CLOCK)**: Enables circular traversal for the clock algorithm with reference bits.

### 6. Error Handling

The program includes comprehensive error handling:

- **Invalid Command Line Arguments**: Displays error message and exits.
- **Invalid Frame Count**: Validates that frame count is numeric.
- **Invalid Replacement Policy**: Checks policy name against valid options.
- **Invalid Input Format**: Validates that all page references are numeric.
- **File Not Found**: Gracefully falls back to STDIN if input file cannot be opened.

## Building and Compilation

The project includes a Makefile for easy compilation:

```bash
cd src
make
```

This will create the `virtualmem` executable.

To clean build artifacts:
```bash
make clean
```

## Usage Examples

### Example 1: Using FIFO with default frames
```bash
./virtualmem -r FIFO -i input.txt
```

### Example 2: Using LFU with 10 frames
```bash
./virtualmem -f 10 -r LFU -i input.txt
```

### Example 3: Using LRU-CLOCK with STDIN input
```bash
./virtualmem -f 8 -r LRU-CLOCK
# Then enter page references when prompted
```

### Example 4: Display help
```bash
./virtualmem -h
```

## Algorithm Selection Guide

Choose an algorithm based on your needs:

- **FIFO**: Simplest to understand, good for educational purposes.
- **LFU**: Best when page access patterns have clear frequency differences.
- **LRU-STACK**: Excellent for workloads with strong temporal locality.
- **LRU-CLOCK**: Good balance of performance and implementation complexity, efficient for large frame counts.
- **LRU-REF8**: Provides enhanced accuracy using recent history, good for varied access patterns.

## Technical Details

- **Language**: C++
- **Compilation**: Requires g++ compiler
- **Dependencies**: Standard C++ libraries, POSIX system calls (`gettimeofday`, `getopt`)
- **Platform**: Unix-like systems (Linux, macOS)



## License

Copyright notice as specified in source files.
