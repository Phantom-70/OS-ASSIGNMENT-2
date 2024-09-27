# OS Assignment 2 - IPC Implementations

This repository contains C code implementations for the Producer-Consumer problem using Inter-Process Communication (IPC) mechanisms. The code addresses three sub-problems with varying numbers of producers and consumers, as well as a bonus section involving namespaces.

## Instructions to Run

1. **Compilation:**
   * Make sure you have a C compiler (like GCC) installed.
   * Use the provided Makefile to compile the code:
     * `make all`  (compiles all sub-problems)
     * `make q_1` (compiles sub-problem 1)
     * `make q_2` (compiles sub-problem 2)
     * `make q_3` (compiles sub-problem 3)
     * `make bonus` (compiles the bonus section - if implemented)
     * `make clean` (removes compiled executables)

2. **Execution:**
   * Run the compiled executables from your terminal:
     * `./q_1`
     * `./q_2`
     * `./q_3`

3. **Output:**
   * The programs will create producer and consumer files (e.g., `prod_1.txt`, `cons_1.txt`) containing the generated characters and vowel counts, respectively.
   * The consumer files will be updated every minute with the running vowel count.
   * After 5 minutes, the programs will terminate, and the final vowel counts will be appended to the consumer files.

## Explanation of Code Files

* `q_1.c`: Implements sub-problem 1 (single producer, single consumer).
* `q_2.c`: Implements sub-problem 2 (single producer, multiple consumers).
* `q_3.c`: Implements sub-problem 3 (multiple producers, multiple consumers).
* `bonus.c`: (Optional) Implementation for the bonus section (namespaces).

## Logic Explanation

### Core Concepts

* **Shared Memory:**  Shared memory segments (`shmid`, `shmid2`, `shmid3`) are created to enable efficient data sharing between the producer and consumer processes.
* **Semaphores:** Semaphores (`full`, `empty`, `mut`) are used for synchronization:
    * `full`:  Counts the number of filled slots in the shared buffer.
    * `empty`: Counts the number of empty slots in the shared buffer.
    * `mut`:  Provides mutual exclusion for critical sections (accessing shared data).
* **Mutex:** A mutex (`my_mutex`) is used to protect the `count_buffer` (vowel count) from concurrent access by multiple consumer threads.
* **Fork and Threads:** The program uses `fork()` to create separate producer and consumer processes. Within each process, multiple threads are created using `pthread_create()` to handle production and consumption concurrently.

### Producer Logic

1. **Initialization:**
   * Each producer thread opens a file (`prod_<thread_number>.txt`) for writing.
   * It allocates a buffer to store characters temporarily before writing them to the file.
   * It keeps track of the last write time to ensure periodic file updates.

2. **Character Generation and Buffering:**
   * In a loop (running for 5 minutes), each producer:
     * Generates a random lowercase alphabet.
     * Waits for an empty slot in the shared buffer using `sem_wait(empty)`.
     * Acquires the mutex (`sem_wait(mut)`) to ensure exclusive access to the shared buffer.
     * Writes the character to the shared buffer and updates the `req_buffer` (write index).
     * Also, stores the character in its local buffer.
     * Releases the mutex (`sem_post(mut)`) and signals that a slot is filled (`sem_post(full)`).
     * Checks if it's time to write to the file (every minute or if the local buffer is full) and flushes the buffer to the file if needed.

3. **Termination:**
   * After 5 minutes, any remaining characters in the local buffer are written to the file.
   * The producer thread exits.
   * The child process (containing producer threads) sends a signal to terminate the parent process (containing consumer threads) and cleans up shared resources.

### Consumer Logic

1. **Initialization:**
   * Each consumer thread opens a file (`cons_<thread_number>.txt`) for writing.
   * It keeps track of the last write time for periodic file updates.

2. **Character Consumption and Vowel Counting:**
   * In an infinite loop, each consumer:
     * Waits for a filled slot in the shared buffer using `sem_wait(full)`.
     * Acquires the mutex (`sem_wait(mut)`).
     * Reads a character from the shared buffer and updates the `req_buffer` (read index).
     * If the character is a vowel, increments its vowel count in the `count_buffer` while holding the `my_mutex` to prevent race conditions.
     * Releases the mutex (`sem_post(mut)`) and signals an empty slot (`sem_post(empty)`).
     * Checks if it's time to write to the file (every minute) and writes the current vowel count if needed.

3. **Termination:**
   * The consumer threads continue running until they receive a termination signal from the producer process.
   * The parent process (containing consumer threads) waits for the child process to finish and then exits.
