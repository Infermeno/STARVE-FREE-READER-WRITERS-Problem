# STARVE-FREE-READER-WRITERS-Problem
In a classical reader-writer problem critical section can be arrived by any number of readers but not alongwith any writer. Also no more than one writer is allowed to access the critical section.

I've described the traditional solution first, then the starvation-free alternative.

### Classical approach

I would be using two binary semaphores(`mutex`,`wrt_mutex`) and one counting semaphore(`readcount`) for classical approach of reader-writers problem.

```cpp
//initialization
mutex=1;
wrt_mutex=1;
int count=0;
```

### Reader
Following is the code for reader in classical solution.

```cpp
wait(mutex);
count++;
if(count==1) wait(wrt_mutex);
signal(mutex);

   //critical section
   
wait(mutex);
count--;
if(count==0) signal(wrt_mutex);
signal(mutex);
```

### Writer
Following is the code for writer in classical solution.

```cpp
wait(wrt_mutex);

// critical section

signal(wrt_mutex);
```

Here `mutex` ensures the mutual exclusion readers while accessing the readcount++. Mutual exclusion of writer and reader for accesing critical section is ensured by `wrt_mutex`. But writer can only be ready to enter critical section only when a reader signals `wrt_mutex` but many readers can access critical section at once therefore writer will starve.

## Starve-Free Solution

In addition to the `mutex` and `wrt_mutex` semaphore,i have used a third binary semaphore `in_mutex` for starvation free approach.
The following is the code for starve free approach.
```cpp
//initialization
mutex=1;
wrt_mutex=1;
in_mutex=1;
count=0;
```

### Reader
The following is the implementation of the reader code using the starve-free approach.

```cpp
wait(in_mutex);
wait(mutex);
count++;
if(count==1) wait(wrt_mutex);
signal(mutex);
signal(in_mutex);

   //critical section
   
wait(mutex);
count--;
if(count==0) signal(wrt_mutex);
signal(mutex);
```

### Writer
The following is the implementation of the reader code using the starve-free approach.

```cpp
wait(in_mutex);
wait(wrt_mutex);

// critical section

signal(wrt_mutex);
signal(in_mutex);
```

Here `in_mutex` solves the problem of starvation for writer. If a write is between two read process then `wait(in_mutex)` in writer code will decrement the value of semaphore `in_mutex` and therefore all other process after write will go to suspend list which can only be awake after a successfull write operation. Therefore the problem of starvation is solved.

