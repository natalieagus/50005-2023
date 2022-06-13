---
title: Deadlock
permalink: /os_notes/week5_deadlock
key: week5_deadlock
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

## System Resources
A system consists of a <span style="color:#f7007f;"><b>finite</b></span> number of resources to be distributed among a number of <span style="color:#f7007f;"><b>competing processes</b></span>. Each type of resource can have several finite instances. Some example include:
* CPU cycles / cores
* I/O devices
* Access to files or memory locations (guarded by locks or semaphores, etc)

A process must <span style="color:#f77729;"><b>request</b></span> a resource before using it and must <span style="color:#f77729;"><b>release</b></span> the resource after using it. A process may <span style="color:#f77729;"><b>request as many</b></span> resources as it requires to carry out its designated task. Obviously, the number of resources requested should not exceed the total number of resources available in the system. 
{:.warning}

In other words, a process cannot request three printers if the system has only two.

Under the normal mode of operation, a process may utilize a resource in only the following sequence:
* <span style="color:#f77729;"><b>Request</b></span>: The process requests the resource. If the request cannot be granted immediately (for example, if the resource is being used by another process), then the requesting process must wait until it can acquire the resource.

* <span style="color:#f77729;"><b>Use</b></span>: The process can operate on the resource (for example, if the resource is a printer, the process can print on the printer).

* <span style="color:#f77729;"><b>Release</b></span>: The process releases the resource.

The request and release of resources may require system calls, depending on who manages the resources.
{:.error}

### Kernel Managed Resources
For each use of a <span style="color:#f77729;"><b>kernel</b></span>-managed resource, the operating system checks to make sure processes who requested these resources has been granted allocation of these resources. Examples are the `request()` and `release()` device, `open()` and `close()` file, and `allocate()` and `free()` memory system calls.

> Some implementation detail: A typical OS manages some kind of system table (data structure) that records whether each resource is <span style="color:#f77729;"><b>free</b></span> or <span style="color:#f77729;"><b>allocated</b></span>. For each resource that is allocated, the table also records the process to which it is allocated. If a process requests a resource that is currently allocated to another process, it can be added to a <span style="color:#f7007f;"><b>queue</b></span> of processes waiting for this resource. 
 
Developers who simply write programs utilising these resources simply make the system calls and need not care about *how* Kernel manages these resources (abstracted).  
{:.warning}

### User Managed Resources
For each use of <span style="color:#f77729;"><b>user</b></span>-managed resources, we can guard them using <span style="color:#f77729;"><b>semaphores</b></span> or <span style="color:#f77729;"><b>mutexes</b></span>. The request and release of semaphores can be accomplished through the `wait()` and `signal()` operations on semaphores, or through `acquire()` and `release()` of a mutex lock. 

> Writing these series of wait() and signal() are defined by developers, and they have to be very careful in writing them else it might result in a deadlock situation. 

# The Deadlock Problem

Deadlock is a situation whereby a set of <span style="color:#f7007f;"><b>blocked</b></span> processes (none can make <span style="color:#f7007f;"><b>progress</b></span>) each holding a resource and waiting to acquire a resource held by another process in the set. 
{:.warning}

The locking tools we learned in the previous chapter *prevents* race condition, but if not implemented properly it can cause <span style="color:#f7007f;"><b>deadlock</b></span>. 

## Example
For example, consider two mutex locks being initialized:
```cpp
pthread_mutex_t first_mutex;
Pthread_mutex_t second_mutex;
pthread_mutex_init(&first_mutex,NULL); 
pthread_mutex_init(&second_mutex,NULL);
```

And two threads doing these work <span style="color:#f7007f;"><b>concurrently</b></span> potentially results in deadlock:
> Can you identify the order of execution that causes deadlock?


```cpp
// Thread 1 Instructions
pthread_mutex_lock(&first_mutex); 
pthread_mutex_lock(&second_mutex); 
/**
* Do some work
*/
pthread_mutex_unlock(&second_mutex); 
pthread_mutex_unlock(&first_mutex);
pthread_exit(0);
/************************************/


// Thread 2 Instructions
pthread_mutex_lock(&second_mutex); 
pthread_mutex_lock(&first_mutex); /**
/**
* Do some work
*/
pthread_mutex_unlock(&first_mutex); 
pthread_mutex_unlock(&second_mutex);
pthread_exit(0);
/************************************/
```

Consider the scenario where Thread 1 acquires `first_mutex` and then suspended, then Thread 2 acquires `second_mutex`. 
* <span style="color:#f77729;"><b>Neither thread will give up</b></span> their currently held `mutex`,
* However they need each other’s mutex lock to <span style="color:#f77729;"><b>continue</b></span>: hence <span style="color:#f7007f;"><b>neither made progress</b></span> and they are in a <span style="color:#f7007f;"><b>deadlock</b></span> situation. 

## Deadlock Necessary Conditions
A deadlock situation *may* arise if the following four conditions hold <span style="color:#f7007f;"><b>simultaneously</b></span>  in a system. These are necessary but not sufficient conditions:
> Read: **simultaneously**, means ALL of them must happen to even have a <span style="color:#f77729;"><b>probability</b></span> of deadlock

1. <span style="color:#f77729;"><b>Mutual exclusion</b></span>
   * At least one resource must be held in <span style="color:#f77729;"><b>a non-sharable mode</b></span>.
   * If another process requests that resource that's currently been held by others, the requesting process must be <span style="color:#f77729;"><b>delayed</b></span> until the resource has been released.

2. <span style="color:#f77729;"><b>Hold and Wait</b></span>
   * A process must be <span style="color:#f77729;"><b>holding</b></span> at least one resource and <span style="color:#f77729;"><b>waiting</b></span> to acquire additional resources that are currently being held by other processes.

3. <span style="color:#f77729;"><b>No preemption</b></span>[^1]
   * Resources can only be <span style="color:#f77729;"><b>released</b></span> only <span style="color:#f77729;"><b>after</b></span> that process has <span style="color:#f77729;"><b>completed</b></span> its task, <span style="color:#f7007f;"><b>voluntarily</b></span> by the process holding it.

4. <span style="color:#f77729;"><b>Circular Wait</b></span>
   * There exists a cycle in the *resource allocation graph* (see next section)

These conditions are necessary but not sufficient, meaning that if all four are present, it is not 100% guaranteed that there is currently a deadlock. 
{:.error}


> Think! Since these conditions are *necessary* for deadlock to happen, <span style="color:#f7007f;"><b>removing</b></span> just one of them <span style="color:#f7007f;"><b>prevents</b></span> deadlock from happening at all. 


[^1]: Preemption:  the act of temporarily interrupting a task being carried out by a process or thread, without requiring its cooperation, and with the intention of resuming the task at a later time. 

## Resource Allocation Graph
Deadlocks can be described more precisely in terms of a directed graph called a<span style="color:#f77729;"><b> system resource-allocation graph</b></span>.

The graph description:
* A set of vertices **V** that is partitioned into two different types of **nodes**: 
  * Active processes (**circle** node)  and 
  * All resource types (**square** node) in the system
* <span style="color:#f77729;"><b>Directed</b></span> edges from:
  * <span style="color:#f77729;"><b>Process to resource</b></span> nodes: process requesting a resource (<span style="color:#f77729;"><b>request edge</b></span>)
  * <span style="color:#f77729;"><b>Resource to process</b></span> nodes: assignment / allocation of that resource to the process (<span style="color:#f77729;"><b>assignment edge</b></span>)
* Resource <span style="color:#f77729;"><b>instances</b></span> within each resource type node (<span style="color:#f77729;"><b>dots</b></span>)

### Example 1
Suppose a system has the following <span style="color:#f77729;"><b>state</b></span>:
<img src="/50005/assets/images/week5/1.png"  class="center_seventy"/>


The resource allocation graph illustrating those states is as follows:
<img src="/50005/assets/images/week5/2.png"  class="center_seventy"/>

### Analysing the Graph
If the graph <span style="color:#f77729;"><b>has no cycle</b></span>: deadlock will <span style="color:#f7007f;"><b>never</b></span> happen.

If there’s *at least* 1 cycle:
* If <span style="color:#f77729;"><b>all</b></span> resources has exactly <span style="color:#f77729;"><b>one</b></span> instance, then <span style="color:#f7007f;"><b>deadlock</b></span> (deadlock necessary and sufficient condition)
* If cycle involves only <span style="color:#f77729;"><b>a set</b></span> of resource types with <span style="color:#f77729;"><b>single</b></span> instance, then <span style="color:#f7007f;"><b>deadlock</b></span> (deadlock necessary and sufficient condition)
* Else if cycle involves a set of resource types with <span style="color:#f7007f;"><b>multiple</b></span> instances, then *maybe* deadlock (we can't say for sure, this is just a necessary but not sufficient condition)

In Example 1 diagram above, the **three** processes are deadlocked (process and resource is shortened as `P` and `R` respectively):
* `P1` needs `R1`, which is currently held by `P2` 
* `P2` needs `R4`, which is currently held by `P3`
* `P3` needs `R2`, which is currently held by either `P1` or `P2` 
> Neither process can give up their currently held resources to allow the continuation of the others resulting in a deadlock. 

### Example 2
Now consider the another system state  below. Although *there are cycles*,  this is <span style="color:#f7007f;"><b>not</b></span> a deadlocked state because `P1` might eventually release `R2` after its done, and `P3` may acquire it and complete. Finally, `P2` may resume to completion after `P3` is done. 

<img src="/50005/assets/images/week5/3.png"  class="center_seventy"/>



