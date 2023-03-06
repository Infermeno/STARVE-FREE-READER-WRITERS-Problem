# STARVE-FREE-READER-WRITERS-Problem
In a classical reader-writer problem critical section can be arrived by any number of readers but not alongwith any writer. Also no more than one writer is allowed to access the critical section.

I've described the traditional solution first, then the starvation-free alternative.

### Classical approach

I would be using two binary semaphores(`mutex`,`wrt`) and one counting semaphore(`readcount`) for classical approach of reader-writers problem.

```cpp
//initialization
mutex=1;
wrt=1;
int readcount=0;
```

### Reader
Following is the code for reader in classical solution.

```cpp
//  READER'S CODE
do{
    // Entry 
    wait(mutex,ID);
    readcount++;
    if(readcount == 1) wait(wrt,ID);
    signal(mutex);

    /**
     * 
     * Critical Section
     * 
    */

   // Exit 
   wait(mutex,ID);
   readcount--;
   if(readcount == 0) signal(wrt);
   signal(mutex);

    // Remainder Section

}while(true);
```
### Writer
Following is the code for writer in classical solution.

```cpp
//  WRITER'S CODE
do{
    // Entry
   wait(wrt,ID);
    
    /**
     * 
     * Critical Section
     * 
    */

   // Exit
   signal(wrt);

   // Remainder Section

}while(true);
```
Here `mutex` 

## Starve-Free Solution

In addition to the `mutex` and `wrt` semaphore,i have used a third binary semaphore `in` for starvation free approach.
The following is the code for starve free approach.

### Semaphores and initialization

```cpp
// INITIALIZATION //
Semaphore *mutex = new Semaphore(1);
Semaphore *in = new Semaphore(1);
Semaphore *wrt = new Semaphore(1);
int readcount = 0;
```
### Reader Implementation (Starve-Free)
The following is the implementation of the reader code using the starve-free approach.

```cpp
//  STARVE-FREE READER CODE
do{
    
    // Entry
    wait(in,ID);
    wait(mutex,ID);
    readcount++;
    if(readcount == 1) wait(wrt,ID);
    signal(mutex);
    signal(in);

    /**
     * 
     * Critical Section
     * 
    */

   // Exit
   wait(mutex,ID);
   readcount--;
   if(readcount == 0)signal(wrt);
   signal(mutex);

   // Remainder Section

}while(true);

```

### Writer Implementation (Starve-Free)
The following is the implementation of the reader code using the starve-free approach.

```cpp
//  STARVE-FREE WRITER CODE
do{

    // Entry
    wait(in,ID);
    wait(wrt,ID);

    /**
     * 
     * Critical Section
     * 
    */

    // Exit
    signal(wrt);
    signal(in);

    // Remainder Section

}while(true);
```
Before anyone (the reader or the writer) may access the `wrt` or before anyone (any reader) can enter the critical part directly, `in` must first be completed. This eliminates the problem of starving as if readers keep coming one after another, then this won't starve the writers like it used to above. In this case, if a writer enters between two readers and even if some readers remain in the critical section, the subsequent reader, if it enters after a writer, would have already acquired the `in` and thus be unable to do so. As a result, after the current readers leave the critical section, the writer who was waiting would be at the head of the`blocked_Queue` for the `wrt` and would subsequently acquire.Therefore starvation issue has been resolved, ensuring that neither the readers nor the writers will starve. 

