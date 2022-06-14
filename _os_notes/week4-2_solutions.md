---
title: Critical Section Solutions
permalink: /os_notes/week4_solutions
key: os-notes-week4_solutions
layout: article
nav_key: os_notes
sidebar:
   nav: os_notes
license: false
aside:
   toc: true
show_edit_on_github: false
show_date: false
---
In the next few sections, we discuss several known solutions to CS problems. They are generally divided into these categories:
* <span style="color:#f77729;"><b>Software Mutex </b></span>Algorithm (purely software only, possible only under restricted conditions, busy waits)
* <span style="color:#f77729;"><b>Hardware</b></span> Supported <span style="color:#f77729;"><b>Spinlocks</b></span> (hardware supported, only 1 process in the CS at a time, busy waits)
* <span style="color:#f77729;"><b>Software Spinlocks and Mutex Locks</b></span>
* <span style="color:#f77729;"><b>Semaphores</b></span> (does not busy wait, generalization of mutex locks -- able to protect two or more identical shared resources) 
* <span style="color:#f77729;"><b>Conditional Variables</b></span> (does not busy wait, wakes up on condition) 

# Software Mutex Algorithm
## Peterson's Solution
Peterson’s solution is a <span style="color:#f77729;"><b>software</b></span>-based approach that solves the CS problem, but two restrictions apply:
* Strictly <span style="color:#f77729;"><b>two</b></span> processes that <span style="color:#f77729;"><b>alternate</b></span> execution in a <span style="color:#f77729;"><b>single</b></span> core between their critical sections and remainder sections (can be generalised to `N` processes with proper data structures, out of syllabus)
* Architectures where `LD` and `ST` are <span style="color:#f77729;"><b>atomic</b></span>[^1] (i.e: executed in 1 clk cycle, or not interruptible)


The solution works by utilizing two shared global variables:
```cpp
int turn;
bool flag[2];
```
* If `turn == i`, process `Pi` is allowed to <span style="color:#f77729;"><b>enter</b></span> the critical section. Similar otherwise.
* If `flag[i] == true`, then process `Pi` is <span style="color:#f77729;"><b>ready</b></span> to enter the critical section.

### Initialisation
We can initialize the `flag` into `false` for all `i`, and set `turn` into <span style="color:#f77729;"><b>arbitrary number</b></span> (`i` or `j` to index two processes `Pj` and `Pi` -- two processes only for our syllabus) in the beginning.

### Algorithm
```cpp
do{
   flag[i] = true;
   turn = j;
   while (flag[j] && turn == j); // this is a while LINE
   // CRITICAL SECTION HERE
   // ...
   flag[i] = false;
   // REMAINDER SECTION HERE
   // ...
}while(true)
```

The algorithm above is the solution for process `Pi`. 
* In the `while`-loop, `Pi` busy waits (means <span style="color:#f77729;"><b>try</b></span> and keep <span style="color:#f77729;"><b>retrying</b></span> until succeed, thus wasting the quanta given to the process), 
* `Pi` will be <span style="color:#f7007f;"><b>stuck</b></span> at the while-line (notice it's NOT a while-loop, there’s a semicolon at the end), for as long as `flag[j] == true` <span style="color:#f7007f;"><b>and</b></span> `turn == j`. 

## Proof of Correctness
To prove that this solution is correct, we need to show that:
* <span style="color:#f77729;"><b>Mutual exclusion</b></span> is preserved.
* The <span style="color:#f77729;"><b>progress</b></span> requirement is satisfied.
* The <span style="color:#f77729;"><b>bounded-waiting</b></span> requirement is met.


We prove that the Peterson’s solution satisfies all <span style="color:#f77729;"><b>three</b></span> properties above by tracing how it works:
* When `Pi` wants to enter the critical section, it will set its own flag; `flag[i]` into `true`. 
* Then, it sets `turn=j` (instead of itself, i). 
> You can think of a process as being <span style="color:#f77729;"><b>polite</b></span> hence requesting the *other one* (`j`) to be executed <span style="color:#f77729;"><b>first</b></span>.

Now two different scenarios might happen.

### Scenario 1: Proceed to CS
`Pi`might break from the `while`-loop under two possible conditions.

<span style="color:#f77729;"><b>Conditon 1:</b></span> `flag[j] == false`, meaning that the other `Pj` is <span style="color:#f7007f;"><b>not ready</b></span> to enter the CS and is <span style="color:#f7007f;"><b>also not in the critical section</b></span>. This ensures <span style="color:#f77729;"><b>mutex</b></span>.

<span style="color:#f77729;"><b>Condition 2:</b></span> OR, IF `flag[j] == true` but `turn == i`. This means the other process `Pj` is also <span style="color:#f77729;"><b>about</b></span> to enter the critical section. No process is in the Critical Section, but it is `Pi`'s turn, so `Pi` gets to enter the CS first (ensuring <span style="color:#f77729;"><b>progress</b></span>). 

### Scenario 2: Busy-wait
`Pi` will be stuck with the `while`-line if `flag[j] == true` <span style="color:#f7007f;"><b>AND</b></span> `turn == j`, meaning that `Pj`is inside the CS. 

This satisfies <span style="color:#f77729;"><b>mutual exclusion</b></span>: `flag[i]` and `flag[j]` are both `true`, but `turn` is an <span style="color:#f77729;"><b>integer</b></span> and cannot be both `i` and `j`. 
* No two processes can simultaneously execute the CS. 

<span style="color:#f77729;"><b>Bounded waiting is guaranteed</b></span>: After `Pj` is done, it will set its own flag: `flag[j] = false`, hence ensuring `Pi` to <span style="color:#f77729;"><b>break</b></span> out of the `while`-line and enter CS in the future. 
* `Pi` only needs to wait a <span style="color:#f7007f;"><b>MAXIMUM</b></span> of 1 cycle before being able to enter the CS.

> You might want to <span style="color:#f77729;"><b>interleave</b></span> the execution of instructions between `Pi` and `Pj`, and convince yourself that this solution is indeed a legitimate solution to the CS problem. Try to also modify some parts: not to use `turn`, set `turn` for `Pi` as itself (`i`) instead of `j`, not to use `flag`, etc and convince yourself whether the 3 <span style="color:#f77729;"><b>correctness</b></span> property for the original Peterson's algorithm still apply. 

## Why Atomic LD/ST is Required?
Not all LD/ST instructions are guaranteed to be atomic, we take this for granted. Consider a scenario of a non-atomic database containing student grades, and initial grade of student I of `60`, and these two actions performed sequentially (but not atomic): 
1. Professor A updates the grade of student I to 90
2. Professor B fetch the grade of student I, and increment it by 2 and submit the copy to student Admin 

By right, the grade of student I should be 92. However, since the database is not atomic, Professor B fetched the old grade of student I (which means the update done by Professor A is not *immediately* reflected in the database). Student I end up having a grade of 62 instead of 92 (got a C instead of an A!). 

What happened? 
> The STORE done by Professor A is not ATOMIC, meaning that what Professor B saw was a "stale" grade, although Professor B's action is done *after* Professor A. Many operations of read and write from a distributed database are not atomic because it causes a slow performance. *Actually* storing the value into the database is done in background to improve interactivity. 

When we <span style="color:#f77729;"><b>check</b></span> for `turn == j`, or `flag[j] == True` we need to be sure that these values that are read are not *stale*. Equivalently, any STORE of `turn` or `flag` value should complete before subsequent attempts to access `turn` and `flag` is done, otherwise it is possible for both processes to enter the CS (or stuck in busy wait). If you're interested, you can read [this](https://harmony.cs.cornell.edu/docs/textbook/spinlock/) article. If not, just accept that Peterson's solution require atomic `LD/ST` 😉. 

# Synchronization Hardware 
Peterson’s solution is a software solution that is not guaranteed to work on modern computer architectures where `LD/ST` might not be atomic (e.g: there are many processors accessing the same location).

We can also solve the CS Problem using hardware solution. All these solutions are based on the premise of <span style="color:#f77729;"><b>hardware locking</b></span> to protect the critical sections. This is significantly different from software locks. The number of hardware locks per system is <span style="color:#f77729;"><b>physically</b></span> limited.  

## Preventing Interrupts 
We can intentionally prevent interrupts[^2] from occurring while a shared variable was being modified. 

This only works in <span style="color:#f77729;"><b>single-core</b></span> systems and is <span style="color:#f7007f;"><b>not</b></span> feasible in multiprocessor environments because:
* Time consuming, need to pass message to all processors
* Affects system clocks
* Decreases efficiency, defeats the purpose of multiprocessors. 

The ability to temporarily inhibit interrupts and ensuring that the currently running process cannot be context switched need to be supported at hardware levels in many modern architectures. If no interrupt can occur during a particular CS, then mutual exclusion is <span style="color:#f77729;"><b>trivial</b></span>.

## Atomic Instructions
We can also implement a set of <span style="color:#f77729;"><b>atomic</b></span> instructions by <span style="color:#f77729;"><b>locking the memory bus</b></span>. Mutual exclusion is therefore <span style="color:#f77729;"><b>trivial</b></span>. 
{:.warning}

This is typically used to provide <span style="color:#f77729;"><b>mutual exclusion</b></span> in symmetric multiprocessing systems. The common instructions are:
* Swap two memory words: `compare_and_swap()`
* Test original value of memory word and set its value: `test_and_set()`

Example:
* Lets say **CPU1** issues `test_and_set()`, the modern DPRAM (dual port ram) makes an internal note by <span style="color:#f77729;"><b>storing</b></span> the address of the memory location in its own dedicated place. 
* If **CPU2** also issues `test_and_set()` to the <span style="color:#f7007f;"><b>same</b></span> location then  DPRAM checks its internal note and notices that CPU1 is accessing it, so it will issue a <span style="color:#f77729;"><b>busy</b></span> interrupt to **CPU2**. 
* **CPU2** might be busy waiting (spinlock) to <span style="color:#f7007f;"><b>retry</b></span> again (until succeeds). This happens at <span style="color:#f77729;"><b>hardware speed</b></span> so it is actually not very long at all. 
* After CPU1 is done, the DPRAM erases the internal note, thus allowing CPU2 to execute its `test_and_set()`. 

Below are the common atomic instructions supported at the hardware level. They are used <span style="color:#f77729;"><b>directly</b></span> by compiler and operating system <span style="color:#f77729;"><b>programmers</b></span> but are also abstracted and exposed as bytecodes and library functions in higher-level languages like C:
* atomic `read` or `write`
* atomic `swap`
* `test-and-set`
* `fetch-and-add`
* `compare-and-swap`
* `load-link`/`store-conditional`


# Software Spinlocks and Mutex Locks
We need hardware support for certain <span style="color:#f77729;"><b>special</b></span> atomic assembly-language instructions like `test-and-set` above. This can be used by application programmers to <span style="color:#f77729;"><b>implement</b></span> software locking without the need to switch to kernel mode or perform context switch (unless otherwise intended). On architectures without such hardware support for atomic operations, a non-atomic locking algorithm like Peterson's algorithm can be used. 
> However, such an implementation may require more memory than a hardware spinlock.

## Spinlocks
A spinlock provides mutual exlusion. It is simply a variable that can be initialized, e.g `pthread_spinlock_t` implemented in C library and then obtained or released using two <span style="color:#f77729;"><b>standard</b></span> methods like `acquire()` and `release()`. An attempt to `acquire()` the lock causes a process or thread trying to acquire it to wait in a loop ("spin") while repeatedly checking whether the lock is available.  
> This is also called <span style="color:#f77729;"><b>busy waiting</b></span>, hence wasting the caller's quantum until the caller acquires the lock and can progress. 

Example implementation of `acquire()` in C library:
```cpp
acquire() {
   /* this test is performed using atomic test_and_set and busy wait for the process' CS, hardware supported */ 
     while (!available); 
     available = false;
}

release(){
	available = true;
}
```

You can create spinlock using C POSIX library: `pthread_spin_lock()`
```cpp
static pthread_spinlock_t spinlock;
pthread_spin_init(&spinlock,0);
pthread_spin_lock(&spinlock); // no context switch, no system call, busy waits if not available
// CRITICAL SECTION ...
pthread_spin_unlock(&spinlock);
// REMAINDER SECTION ...
```

### Busy Waiting
Busy waiting <span style="color:#f77729;"><b>wastes</b></span> CPU cycles -- some other process might be able to use productively, and it affects efficiency tremendously when a CPU is shared among many processes. The spinning caller will utilise 100% CPU time just waiting: repeatedly  checking if a spinlock is available. 
> Does it make much sense to use spinlocks in single-core environment? The spinlock polling is blocking the only available CPU core, and as a result no other process can run. Since no other process can run, the lock won't be unlocked either. 


Nevertheless, spinlocks are mostly useful in places where anticipated waiting time is <span style="color:#f77729;"><b>shorter</b></span> than a quantum and that multicore is present. <span style="color:#f7007f;"><b>No context switch is required</b></span> when a process must wait on a lock, the calling process simply utilise the special assembly instruction. 
{:.warning}

## Mutex Lock
Some other libraries (like C POSIX library) implement another type of lock called <span style="color:#f77729;"><b>mutex lock</b></span> that does not cause busy wait. 

Mutex locks are also supported by special atomic assembly instructions implemented at hardware level, and requires <span style="color:#f7007f;"><b>integration</b></span> with the Kernel Scheduler because it will put the requesting thread/process to <span style="color:#f77729;"><b>sleep</b></span> if the lock has already been acquired by some other thread. 

Example of POSIX Mutex usage:
```cpp
// initialize mutex
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&mutex); // this is  acquire() equivalent, atomic and might invoke kernel scheduler and context switch if mutex not available  
// CRITICAL SECTION HERE ...
pthread_mutex_unlock(&mutex); // this is  release() equivalent, atomic and might invoke kernel scheduler as well to wake up other waiting processes/threads
// REMAINDER SECTION HERE ...
```

The problem with mutexes is that putting threads to sleep and waking them up again are both rather <span style="color:#f7007f;"><b>expensive</b></span> operations, they'll need quite a lot of CPU instructions <span style="color:#f7007f;"><b>overhead</b></span> due to the integration with the Kernel Scheduler. 
{:.warning}

If the CS is short, then the overhead of using mutexes might be <span style="color:#f77729;"><b>more</b></span> than the time taken to "spin" when using a spinlock. It is up to the programmer which one to use for their purposes. 

# Semaphores
The semaphore can be seen as a <span style="color:#f77729;"><b>generalised mutex lock</b></span>. It provides <span style="color:#f77729;"><b>mutual exclusion</b></span>. 

It is implemented at the <span style="color:#f77729;"><b>kernel</b></span> level, meaning that its execution requires the calling process to change into the kernel mode via trap (SVC) semaphore instructions. 

This is a high-level <span style="color:#f77729;"><b>software</b></span> solution that relies on <span style="color:#f7007f;"><b>synchronization hardware</b></span> (like those special atomic instructions), and is considered a more <span style="color:#f77729;"><b>robust</b></span> tool than mutex lock. 
> Peterson’s solution and the hardware-assisted solutions all require busy waiting. However using semaphore, you can express a solution to the CS problem without busy waiting. 

## Definition
Semaphore is defined as:
* An <span style="color:#f77729;"><b>integer variable</b></span> that is maintained in the kernel, initialized to a certain value that represents the number of <span style="color:#f77729;"><b>available</b></span> resources for the sharing processes
* Is accessed only through two standard <span style="color:#f77729;"><b>atomic</b></span> System Call operations: `wait()` and `signal()`.

Since calls to `signal()` or `wait()` for semaphores must be performed atomically, they are often implemented using one of the synchronization hardware mechanisms (to support multiple cores synchronization) or software mutual exclusion algorithms such as Peterson’s algorithm, when restriction applies. 

## Usage
Semaphore can be used in two ways:
* <span style="color:#f77729;"><b>Binary</b></span> semaphore: integer value can be a `0` or `1`. 
* <span style="color:#f77729;"><b>Counting</b></span> semaphore: integer can take any value between `0` to `N`.

## Implementation
How semaphores avoid busy waiting (well, *mostly*): it integrates implementation with CPU scheduler, hence the need to make system calls throughout.  It will put the calling process that attempts to `acquire()` on suspension if the semaphore is not currently available. 

There are two common CPU scheduling operations (System Calls): `block()` and `wakeup()`. Both are used to implement `wait()` and `signal()` semaphore functions. 
* Whenever the process/thread needs to `wait()`, it <span style="color:#f77729;"><b>reduces</b></span> the semaphore and if the current semaphore value is negative, it blocks itself.
* It is added to a <span style="color:#f77729;"><b>waiting queue</b></span> of processes/threads associated with that semaphore. This is done through the system call `block()` on that process. 
```cpp
wait(semaphore *S)
{  S->value--;
   if (S->value < 0)
   {
       add this process to S->list; // this will call block() 
   }
}
```
* When a process is completed it calls a `signal()` function and one process in the queue is resumed, by using the `wakeup()` system call.
```cpp
signal(semaphore *S)
{
   S->value++;
   if (S->value <= 0)
   {
       remove a process P from S->list;
       wakeup(P);
   }
}
```

Further notes about the above simple implementation of Semaphore `wait` and `signal` System Calls:
* Semaphore values may be <span style="color:#f77729;"><b>negative</b></span>, but this is typically <span style="color:#f77729;"><b>hidden</b></span> from user
  * On the surface, semaphore values are never negative under the classical definition of semaphores with busy waiting. 
* If the semaphore value is <span style="color:#f77729;"><b>negative</b></span>, <span style="color:#f7007f;"><b>its magnitude</b></span> is the number of processes waiting on that semaphore
* The list (a queue of processes waiting to acquire the semaphore) can be easily implemented by a link field in each process control block (PCB). 
  * Each semaphore data structure for example, can contain an <span style="color:#f77729;"><b>integer</b></span> value and a <span style="color:#f77729;"><b>pointer to a list of PCBs.</b></span>

Note that semaphore implementation may vary between different libraries, but the idea remains the same. 
{:.warning}

## Circular Dependency?
How can we implement `signal()` and `wait()` atomically <span style="color:#f77729;"><b>without</b></span> busy waiting, if it relies on synchronization hardware in multiprocessor systems or even basic software mutex algorithms (e.g: if on uniprocessor systems) that <span style="color:#f77729;"><b>requires</b></span> busy waiting? 
> The answer is that semaphore <span style="color:#f7007f;"><b>DOES NOT</b></span> completely eliminates busy waiting, 

Specifically: it <span style="color:#f7007f;"><b>ONLY busy waits in the critical section of semaphore</b></span> function itself: `wait()`, `signal()` that is relatively <span style="color:#f7007f;"><b>SHORT</b></span> if implemented properly 
> We do NOT busy-wait in the CS of the program itself (which can be considerably longer). 

In practice, the critical section of the `wait()` and `signal()` implementation of the semaphore in the library is almost never occupied (meaning it’s rare that two processes or threads are making the same wait and signal call on the same semaphore), and busy waiting occurs rarely, or if it does happen, it happens for only a short time. 
{:.warning}

In this specific situation, busy waiting is <span style="color:#f77729;"><b>completely acceptable</b></span> and still remains efficient. 

## Applying Semaphore to MPC Problem
Now that we know how semaphore works, it’s useful to think about how they can be applied to tackle the <span style="color:#f77729;"><b>multiple</b></span> <span style="color:#f77729;"><b>producer-consumer</b></span>(MPC in short) problem that we analyze earlier in this section above. 

The pseudocode below illustrates the idea on how the semaphore can be used to replace `counter`, and <span style="color:#f77729;"><b>protect</b></span> the consumer/producer code against multiple (more than 1 of each) consumer/producer threads racing to execute their respective instructions, and sharing resources:
* write index `in` (shared among consumer processes)
* read index `out` (shared among producer processes)
* `char buf[N]` (shared among all processes)
* Two semaphores: one binary and one counting semaphore

Shared resources:
```cpp
char buf[N];
int in = 0; int out = 0;

semaphore chars = 0; 
semaphore space = N;
semaphore mutex_p = 1; 
semaphore mutex_c = 1;
```

Producer program:
```cpp
void send (char c){
   wait(space);
   wait(mutex_p);

   buf[in] = c;
   in = (in + 1)%N;

   signal(mutex_p);
   signal(space);
}
```

Consumer program:
```cpp
char rcv(){
   char c;
   wait(chars);
   wait(mutex_c);

   c = buf[out];
   out = (out+1)%N;

   signal(mutex_c);
   signal(chars);
}
```

# Conditional Variables
Conditional variables allow a process or thread to wait for completion of a given <span style="color:#f7007f;"><b>event</b></span> on a particular object (some shared state, data structure, anything).  It is used to <span style="color:#f77729;"><b>communicate</b></span> between processes or threads when certain conditions become `true`.
> The "event"  is the *change* in state of some condition that thread is interested in. Until that is satisfied, the process waits to be awakened later by a signalling process/thread (that actually <span style="color:#f77729;"><b>changes</b></span> the condition).

Conditional variables are and <span style="color:#f77729;"><b>should</b></span> always be implemented with <span style="color:#f77729;"><b>mutex</b></span> locks. When implemented properly, they provide <span style="color:#f7007f;"><b>condition synchronization</b></span>. 
{:.error}

For example, we can initialize a mutex guarding certain CS:
```cpp
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```

And in this example, we assume that a particular CS in Process/Thread 2 cannot be executed if `bool x == false`. Therefore we can create a condition to represent this:
```cpp
pthread_cond_t cond_x = PTHREAD_COND_INITIALIZER;
```

Now consider Process/Thread 1 instructions:
```cpp
pthread_mutex_lock(&mutex);
// CRITICAL SECTION
// ...
cond_x = true;
pthread_cond_signal(&cond);
pthread_mutex_unlock(&mutex);
```

...  and Process/Thread 2 instructions:
```cpp
pthread_mutex_lock(&mutex);
while (cond_x == false){
   pthread_cond_wait(&cond_x, &mutex);  // yields mutex, sleeping
}
// CRITICAL SECTION, can only be executed iff cond_x == true
// ...
pthread_mutex_unlock(&mutex);
```

It is clear that the `mutex` guards the CS. However, Process 2 should <span style="color:#f77729;"><b>not</b></span> proceed to its CS if `cond_x` is `false` in this example. 

Process 2 can be put to sleep by calling `condition_wait()`; the implementation of this function is integrated into the process scheduler which does the following:
* It <span style="color:#f77729;"><b>gives</b></span> up the mutex <span style="color:#f77729;"><b>AND</b></span>
* <span style="color:#f77729;"><b>Sleeps</b></span>, will not busy wait

When Process 1 has set the variable `cond_x` into `true`, it signals Process 2 to <span style="color:#f77729;"><b>continue</b></span> before <span style="color:#f77729;"><b>giving</b></span> up the mutex. 
* It is important to call `wait()` after acquiring the mutex and checking the condition.
  * You cannot use `unlock()` to release the mutex. 
* When Process 2 `waits` and eventually sleep, it will <span style="color:#f77729;"><b>give</b></span> up the lock so as to allow other threads needing the lock to proceed. 
* When Process 2 has woken up from the `wait()`, it <span style="color:#f77729;"><b>automatically</b></span> reacquires the mutex lock 
  * If Process 1 hasn’t given up the lock, Process 2 will not execute although `cond_x` has been fulfilled

This is crucial because it will re-check the state of `cond_x` again before continuing to the CS. It is also crucial to re-check `cond_x` before continuing to the CS. <span style="color:#f77729;"><b>Why?</b></span>
{:.error}
  
## Final Thoughts
A conditional variable is effectively a <span style="color:#f77729;"><b>signalling</b></span> mechanism under the context of a given mutex lock. With mutex lock alone, we cannot easily block a process out of its CS based on any <span style="color:#f77729;"><b>arbitrary condition</b></span> even when the mutex is <span style="color:#f77729;"><b>available</b></span>.

# Appendix: Sample C Code
## Mutex
In the example below, we attempt to increase a shared variable `counter`, guarded by a `mutex` to prevent race conditions.

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void *functionC();
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int counter = 0;

int main()
{
  int rc1, rc2;
  pthread_t thread1, thread2;

  /* Create independent threads each of which will execute functionC */

  if ((rc1 = pthread_create(&thread1, NULL, &functionC, NULL)))
  {
     printf("Thread 1 creation failed: %d\n", rc1);
  }

  if ((rc2 = pthread_create(&thread2, NULL, &functionC, NULL)))
  {
     printf("Thread 2 creation failed: %d\n", rc2);
  }

  // Main thread waits until both threads have finished execution

  pthread_join(thread1, NULL);
  pthread_join(thread2, NULL);

  return 0;
}

void *functionC()
{
  pthread_mutex_lock(&mutex);
  counter++;
  printf("Counter value: %d\n", counter);
  pthread_mutex_unlock(&mutex);
}
```

## Condition Variables
Consider a main function with a shared variable `count`, two `mutexes` and one `condition`: 

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

pthread_mutex_t count_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t condition_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t condition_cond = PTHREAD_COND_INITIALIZER;

void *functionCount1();
void *functionCount2();
int count = 0;
#define COUNT_DONE 10
#define COUNT_HALT1 3
#define COUNT_HALT2 6

int main()
{
  pthread_t thread1, thread2;

  pthread_create(&thread1, NULL, &functionCount1, NULL);
  pthread_create(&thread2, NULL, &functionCount2, NULL);
  pthread_join(thread1, NULL);
  pthread_join(thread2, NULL);

  return 0;
}
```

Suppose an example where we Thread 1 running `functionCount1()` is to be <span style="color:#f77729;"><b>halted</b></span> whenever the `count` value is between 3 (`COUNT_HALT1`) and 6 (`COUNT_HALT2`). 
* Otherwise, either thread can increment the counter. 
* We can use the condition variable and condition wait to ensure this behavior in `functionCount1()`:

```cpp
void *functionCount1()
{
  for (;;) // equivalent to while(true)
  {
     pthread_mutex_lock(&count_mutex);
     while (count >= COUNT_HALT1 && count <= COUNT_HALT2)
     {
        pthread_cond_wait(&condition_cond, &count_mutex);
     }

     count++;
     printf("Counter value functionCount1: %d\n", count);
     pthread_mutex_unlock(&count_mutex);

     if (count >= COUNT_DONE)
        return (NULL);
  }
}
```

We can then use `cond_signal` in `functionCount2()` executed by Thread 2:
```cpp
void *functionCount2()
{
  for (;;) // equivalent to while(true)
  {
     pthread_mutex_lock(&count_mutex);
     if (count < COUNT_HALT1 || count > COUNT_HALT2)
     {
        pthread_cond_signal(&condition_cond);
     }
     count++;
     printf("Counter value functionCount2: %d\n", count);
     pthread_mutex_unlock(&count_mutex);

     if (count >= COUNT_DONE)
        return (NULL);
  }
}
```
## Producer-Consumer Problem
In this sample, we try to tackle the single producer single consumer problem with counting semaphore. The shared resources will be an integer array named `buffer` in this example, of size `10`. 

The idea is to ensure that producer <span style="color:#f77729;"><b>does not overwrite</b></span> unconsumed values, and to ensure that consumer <span style="color:#f77729;"><b>does not consume anything before</b></span> producer puts anything new into the buffer. 

Main process to initialise Producer and Consumer Threads:
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>
#include <fcntl.h>


#define RESOURCES 10
#define REPEAT 100

sem_t *blank_space;
sem_t *content;

int buffer[RESOURCES];
int write_index = 0;
int read_index = 0;

int main()
{
   // instantiate named semaphore, works on macOS
   // sem_t *sem_open(const char *name, int oflag,
   //                   mode_t mode, unsigned int value);
   // @mode: set to have R&W permission for owner, group owner, and other user
   blank_space = sem_open("blank_space", O_CREAT,
                          S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH,
                          RESOURCES);
   printf("%p \n", (void *)blank_space);
   if (blank_space == (sem_t *)SEM_FAILED)
   {
       printf("Sem Open Failed.\n");
       exit(1);
   }
   content = sem_open("content", O_CREAT,
                      S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH,
                      0);
   printf("%p \n", (void *)content);
   if (content == (sem_t *)SEM_FAILED)
   {
       printf("Sem Open Failed.\n");
       exit(1);
   }

   pthread_t producer, consumer;
   pthread_create(&producer, NULL, producer_function, NULL);
   pthread_create(&consumer, NULL, consumer_function, NULL);

   printf("Joining threads\n");
   pthread_join(producer, NULL);
   pthread_join(consumer, NULL);

   // if you don't destroy, it persists in the system
   // run the command: ipcs -s
   // to remove: ipcrm -s <sem_id>
   sem_unlink("blank_space");
   sem_unlink("content");
   return 0;
}
```

Producer Thread instruction:
```cpp
void *producer_function(void *arg)
{
   for (int i = 0; i < REPEAT; i++)
   {
       // wait
       sem_wait(blank_space);
       // write to buffer
       buffer[write_index] = i;
       // advance write pointer
       write_index = (write_index + 1) % RESOURCES;
       // signal
       sem_post(content);
   }

   return NULL;
}
```

Consumer Thread instruction:
```cpp
void *consumer_function(void *arg)
{
   for (int i = 0; i < REPEAT; i++)
   {
       // wait
       sem_wait(content);
       // read from buffer
       int value = buffer[read_index];
       printf("Consumer reads: %d \n", value);
       // advance write pointer
       read_index = (read_index + 1) % RESOURCES;
       // signal
       sem_post(blank_space);
   }

   return NULL;
}
```

Paste the two functions above before `main()`. After you compile and run the code, you should have an output as such where consumer thread nicely prints out the numbers put into the buffer by producer in sequence (and stops at 100): 

<img src="/50005/assets/images/week4/2.png"  class="center_seventy"/>

[^1]: 
	An operation acting on shared memory is atomic  if it completes in a single step relative to other threads. For example, when an atomic store is performed on a shared variable, no other thread/process can observe the modification half-complete.  

[^2]: 
	You know this as a non-preemptive  approach, and some kernels are non-preemptive (non-interruptible) and therefore will not face the race condition in the kernel level itself.
