# STARVE-FREE-READER-WRITERS-Problem

A classical Reader-Writer Problem is a situation where a data structure can be read and modified simultaneously by concurrent threads.The vital section is only accessible by one Writer at a time. Any amount of Readers can reach the critical area when there is no active Writer. A mutually exclusive object, or mutex, is used to give concurrent threads mutually exclusive access to a certain important data structure. Mutex is an implementation of semaphore, a more general synchronisation concept. A simple image of a semaphore is a positive number which allows increment by one and decrement by one operations. If a decrement operation is invoked by a thread on a semaphore whose value is zero, the thread blocks until another thread increments the semaphore.

I've described the traditional solution first, then the starvation-free alternative.

## Used Data Structures: 

Semaphores are used to deal with the issue of process synchronization. The list of blocked processes is stored in a Queue (FIFO structure) that is part of the semaphore, connected to a critical section. The process is blocked as it enters the "blocked_Queue," and it is only unblocked when a process in front of it signals a semaphore. The code below is implementation of the same.

```cpp
// SEMAPHORE //

struct PCB{
    PCB *next;
    int ID;
    bool state=true;
    // The state determines whether it is blocked(false) or active(true).
    // All the other details of a process that a PCB contains
    .
    .
    .
    .
    .
}

struct blocked_Queue{

    Process *front,*rear;
    blocked_Queue(){
        front = nullptr;
        rear = nullptr;
    }
    
    //pushing a pcb at rear end of queue
    void push(int id){
        Process *pcb=new Process();
        pcb->ID=id;
        if(rear == nullptr){
            front = pcb;
            rear = pcb;
        }
        else{
            rear = pcb;
            rear->next = pcb;
        }
    }

    //poping a PCB from the front end of the queue
    Process *pop(){
        if(front == nullptr){
            return nullptr;
        }
        else{
            Process *pcb = front;
            front = front->next;
            if(front == nullptr){
                rear = nullptr;
            }
            return pcb;
        }
    }

};

struct Semaphore {
    int semaphore = 1;
    //queue to store the waiting processes
    blocked_Queue* blockedqueue = new blocked_Queue(); 
    Semaphore(int n) {
        semaphore = n;
        // number of resources available of that type is n
    }

};
// If all resources are busy, push the process into the waiting queue and 'block' it
void wait(Semaphore *q, int id) {
    --q->semaphore;
    if(q->semaphore < 0) {
        q->blockedqueue->push(id);
    }
}
//Pop the process from the waiting queue if there are any processes awaiting execution.
void signal(Semaphore* p) {
    ++p->semaphore;
    if(p->semaphore <= 0) {
        process* process_next = p->blockedqueue->pop();
        wakeUp(process_next);    
    }
}
void wakeUp(process* q) {
    q->state = true;
}
```

### Initialization of Semaphores

I would be using two binary semaphores(`mutex`,`wrt`) and one counting semaphore(`readcount`) for classical approach of reader-writers problem.

```cpp
// INITIALIZATION //
Semaphore *mutex = new Semaphore(1);
Semaphore *wrt = new Semaphore(1);
int readcount = 0;
```

## Classical Solution

There are two mutex locks implemented using Semaphores namely mutex and wrt. Mutual exclusion of reader is ensured by mutex while accessing the variable counter. wrt makes sure that each writer has exclusive access to the shared memory resource.

### Reader Implementation (Classical)
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
### Writer Implementation (Classical)
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
In this case, all other readers are waiting on mutex if a reader is waiting for a writer process to signal the wrt. All readers can execute read operations simultaneously after the writer process signals the mutex. All writers are paused on wrt until all readers have finished, which results in starvation.

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

