---
title: Memory and Process Management
permalink: /os_notes/week1_mpm
key: os-notes-week1_mpm
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

The Kernel has to <span style="color:#f77729;"><b>manage</b></span> all memory devices in the system (disk, physical memory, cache) so that they can be shared among many other running user programs. The hierarchical storage structure requires a concrete form of memory management since the same data may appear in different levels of storage system. 


## Virtual Memory Implementation
The kernel implements the Virtual Memory. It has to:
1. <span style="color:#f77729;"><b>Support demand paging</b></span> protocol
2. <span style="color:#f77729;"><b>Keep track</b></span> of which parts of memory are currently being used and by whom
3. <span style="color:#f77729;"><b>Decide</b></span> which processes and data to move into and out of memory
4. <span style="color:#f77729;"><b>Mapping</b></span> files into process address space
5. <span style="color:#f77729;"><b>Allocate</b></span> and <span style="color:#f77729;"><b>deallocate</b></span> memory space as needed
   * If RAM is full, <span style="color:#f77729;"><b>migrate</b></span> some contents (e.g: least recently used) onto the <span style="color:#f77729;"><b>swap space</b></span> on the disk
6. <span style="color:#f77729;"><b>Manage</b></span> the <span style="color:#f7007f;"><b>pagetable</b></span> and any operations associated with it.
   
CPU caches are managed entirely by <span style="color:#f7007f;"><b>hardware</b></span> (cache replacement policy, determining cache `HIT` or `MISS`, etc). Depending on the cache hardware, the kernel may do some initial setup (caching policy to use, etc). 
{:.info}

## Configuring the MMU
The MMU (Memory Management Unit) is a computer <span style="color:#f7007f;"><b>hardware</b></span> component that primarily handles translation of virtual memory address to physical memory address. It relies on data on the system's RAM to operate: e.g utilise the <span style="color:#f77729;"><b>pagetable</b></span>. The Kernel sets up the page table and determine rules for the address mapping. 

Recall from 50.002 that the CPU always operates on virtual addresses (commonly <span style="color:#f77729;"><b>linear</b></span>, `PC+4` unless branch or JMP), but they're translated to physical addresses by the MMU. The Kernel is aware of the translations and it is solely responsible to <span style="color:#f7007f;"><b>program</b></span> (configure) the MMU to perform them. 
{:.info}

## Cache Performance

<span style="color:#f7007f;"><b>Caching</b></span> is an important principle of computer systems. We perform the <span style="color:#f77729;"><b>caching algorithm</b></span> each time the CPU needs to execute a new piece of instruction or fetch new data from the main memory. Remember that cache is a <span style="color:#f7007f;"><b>hardware</b></span>, built in as part of the CPU itself in modern computers as shown in the figure below:

<img src="/50005/assets/images/week1/14.png"  class="center_seventy"/>

Note that “DMA” means direct memory access from the device controller onto the RAM[^5] (_screenshot taken from SGG book_).

Data transfer from disk to memory and vice versa is usually controlled by the **kernel** (requires context switching), while data transfer from CPU registers to CPU cache[^6] is usually a <span style="color:#f7007f;"><b>hardware function</b></span> without intervention from the kernel. There is too much <span style="color:#f7007f;"><b>overhead</b></span> if the kernel is also tasked to perform the latter. 

Recall that the Kernel is responsible to program and set up the cache and MMU hardware, and manage the entire virtual memory. Kernel memory management routines are <span style="color:#f77729;"><b>triggered</b></span> when processes running in user mode encounter <span style="color:#f77729;"><b>page-fault</b></span> related interrupts. Careful <span style="color:#f7007f;"><b>selection</b></span> of the page size and of a replacement policy can result in a greatly increased performance. 


Given a cache <span style="color:#f77729;"><b>hit</b></span> ratio $\alpha$, cache <span style="color:#f77729;"><b>miss</b></span> access time $\epsilon$, and cache <span style="color:#f77729;"><b>hit</b></span> access time $\tau$, we can compute the cache <span style="color:#f7007f;"><b>effective access time as</b></span>:

$$\alpha \tau + (1-\alpha) \times \epsilon$$


# Process Management  {#process-management-to-support-multiprogramming-and-time-sharing-feature}

The Kernel is also responsible for managing all processes in the system and support <span style="color:#f7007f;"><b>multiprogramming</b></span> and <span style="color:#f7007f;"><b>timesharing</b></span> feature. 

## Multiprogramming
Multiprogramming is a concept that is needed for <span style="color:#f77729;"><b>efficiency</b></span>. Part of the kernel code is responsible to schedule processes in a computer system. 

A kernel that supports multiprogramming increases <span style="color:#f77729;"><b>CPU utilization</b></span> by organizing jobs (code and data) so that the CPU always has one to execute. 
{:.info}


The reason for the need of multiprogramming are as follows:
* <span style="color:#f7007f;"><b>Single users must be prevented from keeping CPU and I/O devices busy at all times.</b></span>
    * Since the clock cycles of a general purpose CPU is very fast (in Ghz), we don't actually need 100% CPU power in most case
    * It is often *too fast* to be dedicated for just one program for the entire 100% of the time
    * Hence, if multiprogramming is not supported and each process has a fixed quantum (time allocated to execute), then the CPU might spend most of its time <span style="color:#f7007f;"><b>idling</b></span>.
* The kernel must organise jobs (code and data) <span style="color:#f77729;"><b>efficiently</b></span> so CPU always has one to execute:
  * A <span style="color:#f77729;"><b>subset</b></span> of total jobs in the system is kept in memory + swap space of the disk. 
    * Remember: **Virtual memory** allows execution of processes not completely in memory. 
  * One job is selected per CPU and run by the scheduler 
  * When a particular job has to wait (for I/O for example), context switch is performed.
    * For instance, Process A asked for user `input()`, enters Kernel Mode via supervisor call
    * If there's no input, Process A is <span style="color:#f77729;"><b>suspended</b></span> and <span style="color:#f77729;"><b>context switch</b></span> is performed (instead of returning back to Process A)
    * If we return to Process A, since Process A cannot progress without the given input, it will invoke another `input()` request again -- again and again until the input is present. This will waste so much resources

## Timesharing

Multiprogramming allows for <span style="color:#f7007f;"><b>timesharing</b></span>.

Definition of timesharing: context switch that’s performed so <span style="color:#f77729;"><b>rapidly</b></span> that users still see the system as interactive and seemingly capable to run multiple processes despite having limited number of CPUs.
{:.info}

### Context switch

Context switch is the routine of <span style="color:#f77729;"><b>saving</b></span> the state of a process, so that it can be <span style="color:#f77729;"><b>restored</b></span> and <span style="color:#f77729;"><b>resumed</b></span> at a later point in time. For more details on what this `state` comprised of, see Week 2 notes. 

## Further Consideration
Multiprogramming alone <span style="color:#f7007f;"><b>does not</b></span> necessarily provide for user interaction with the computer system. For instance, even though the CPU has <span style="color:#f77729;"><b>something</b></span> to do, it doesn't mean that it *ensures* that there's always interaction with the user, e.g: a CPU can run background tasks at all times (updating system time, monitoring network traffic, etc) and not run currently opened user programs like the Web Browser and Telegram. 

Timesharing is the <span style="color:#f77729;"><b>logical extension of multiprogramming</b></span>. It results in an <span style="color:#f77729;"><b>interactive</b></span> computer system, which provides <span style="color:#f77729;"><b>direct</b></span> communication between the user and the system.
{:.info}



## Process vs Program {#process-vs-program}

<span style="color:#f7007f;"><b>A process is a program in execution</b></span>. It is a unit of work within the system, and a process has a <span style="color:#f77729;"><b>context</b></span> (regs values, stack, heap, etc) while a program does not. 

A program is a _passive entity_, just lines of instructions while a process is an _active entity_, with its state changing over time as the instructions are executed. 
{:.info}

A program resides on _disk_ and isn’t currently used. It <span style="color:#f7007f;"><b>does not</b></span> take up <span style="color:#f77729;"><b>resources</b></span>: CPU cycles, RAM, I/O, etc while a process need these resources to run. 
* When a process is <span style="color:#f77729;"><b>created</b></span>, the kernel allocate these resources so that it can begin to execute. 
* When it <span style="color:#f77729;"><b>terminates</b></span>, the resources are freed by the kernel — such as RAM space is freed, so that other processes may use the resources. 
* A typical general-purpose computer runs multiple processes at a time. 

If you are interested to find the list of processes in your Linux system, you can type `top` in your terminal to see all of them in real time. A single CPU achieves concurrency by <span style="color:#f77729;"><b>multiplexing</b></span> (perform rapid context switching) the executions of many processes. 
{:.info}



### Kernel is not a process by itself

It is easy to perhaps <span style="color:#f7007f;"><b>misunderstand</b></span> that the kernel is a process. 
{:.warning}

The kernel is <span style="color:#f7007f;"><b>not</b></span> a process in itself (albeit there are kernel _threads_, which runs in kernel mode from the beginning until the end, but this is _highly specific, e.g: in Solaris OS_). 

* For instance, I/O handlers are not processes. They do not have <span style="color:#f77729;"><b>context</b></span> (state of execution in registers, stack data, heap, etc) like normal processes do. * They are simply a piece of **instructions** that is written to **handle** certain events.


You can think of a the kernel instead as made up of just :
* Instructions, data:
  * Parts of the kernel deal with memory management, parts of it with scheduling portions of itself (like drivers, etc.), and parts of it with scheduling processes.
* Much like a state-machine that user-mode processes executes to complete a particular <span style="color:#f77729;"><b>service</b></span> and then return to its own instruction
  * User processes running the kernel code will have its mode changed to <span style="color:#f77729;"><b>Kernel Mode</b></span>
  * Once it returns from the handler, its mode is change dback to <span style="color:#f77729;"><b>User Mode</b></span>

Hence, the statement that the <span style="color:#f77729;"><b>kernel is a program that is running at all times</b></span> is technically true because the kernel <span style="color:#f7007f;"><b>IS</b></span> <span style="color:#f7007f;"><b>part of each process</b></span>. Any process may switch itself into Kernel Mode and perform system call routines (software interrupt), or forcibly switched to Kernel Mode in the event of hardware interrupt. 

The Kernel <span style="color:#f7007f;"><b>piggybacks</b></span> on any process in the system to run. 
{:.info}
* If there are no system calls made, and no hardware interrupts, the kernel does nothing at this instant since it is not entered, and there’s nothing for it to do.




### Process Manager {#process-manager}

The **process manager** is part of the kernel code. It is part of the the **scheduler** subroutine, may be called either when there’s timed interrupt by the timed interrupt handler or trap handler when there’s system call made. It manages and keeps track of the system-wide **process table**, a data structure containing all sorts of information about current processes in the system. You will learn more about this in Week 3.  


The process manager is responsible for the following tasks:
1. <span style="color:#f77729;"><b>Creation</b></span> and <span style="color:#f77729;"><b>termination</b></span> of both user and system processes
2. <span style="color:#f77729;"><b>Pausing</b></span> and <span style="color:#f77729;"><b>resuming</b></span> processes in the event of **interrupts** or **system calls**
3. <span style="color:#f77729;"><b>Synchronising</b></span> processes and provides <span style="color:#f77729;"><b>communications</b></span> between virtual spaces (Week 4  materials)
4. Provide mechanisms to handle deadlocks (Week 5 materials) 


# Providing Security and Protection {#providing-security-and-protection}

The operating system kernel provides **protection** for the computer system by providing a mechanism for <span style="color:#f7007f;"><b>controlling access</b></span> of processes or users to resources defined by the OS.

This is done by identifying the <span style="color:#f7007f;"><b>user</b></span> associated with that process:

1. Using user identities (user IDs, security IDs): name and associated number among all numbers
2. User ID then associated with all files, processes of that user to determine <span style="color:#f7007f;"><b>access control</b></span>
3. For shared computer: group identifier (group ID) allows set of users to be defined and controls managed, then also associated with each process, file

You will learn more about this in Week 6 and during Lab 2. During Lab 2, you will learn about <span style="color:#f7007f;"><b>Privilege Escalation</b></span>: an event when a user can change its ID to another effective ID with more rights. You may read on how this can happen in Linux systems [here](https://linux-audit.com/understanding-linux-privilege-escalation-and-defending-against-it/). 

Finally, the Kernel also provides <span style="color:#f7007f;"><b>defensive security measures</b></span> for the computer system: protecting itself against internal and external attacks via firewalls, encryption, etc. There’s a huge range of security issues when a computer is connected in a network, some of them include denial-of-service, worms, viruses, identity theft, theft of service. You will learn more about this in Week 10. 


# Appendix 1: Multiprocessor System {#multiprocessor-system}

Unlike single-processor systems that we have learned before, multiprocessor systems have two or more processors in close communication, sharing the computer bus and sometimes the clock, memory, and peripheral devices.

Multiprocessor systems have three main advantages:
1. Increased <span style="color:#f7007f;"><b>throughput</b></span>: we can execute many processes in parallel
2. <span style="color:#f7007f;"><b>Economy of scale</b></span>: multiprocessor system is cheaper than multiple single processor system
3. Increased <span style="color:#f7007f;"><b>reliability</b></span>: if one core fails, we still have the other cores to do the work


## Symmetric Architecture
There are different architectures for multiprocessor system, such as a <span style="color:#f77729;"><b>symmetric</b></span> architecture — we have multiple CPU chips in a computer system:

<img src="/50005/assets/images/week1/15.png"  class="center_fifty"/>


Notice that each processor has its own set of registers, as well as a private or local cache; however, all processors share the <span style="color:#f77729;"><b>same</b></span> physical memory. This brings about design issues that we need to note in symmetric architectures:

1. Need to carefully control I/O o ensure that the data reach the <span style="color:#f77729;"><b>appropriate</b></span> processor
2. Ensure load <span style="color:#f77729;"><b>balancing</b></span>:  avoid the scenario where one processor may be sitting idle while another is overloaded, resulting in inefficiencies
3. Ensure cache coherency: if a process supports multicore, makes sure that the data integrity spread among many cache is maintained.

## Multi-Core Architecture
Another example of a symmetric architecture is to have **multiple cores on the <span style="color:#f77729;"><b>same</b></span> chip** as shown in the figure below:

<img src="/50005/assets/images/week1/16.png"  class="center_fourty"/>

This carries an advantage that on-chip communication is <span style="color:#f77729;"><b>faster</b></span> than across chip communication. However it requires  a more delicate hardware design to place multiple cores on the same chip. 


## Asymmetric Multiprocessing
In the past, it is not so easy to add a second CPU to a computer system when operating system had commonly been developed for single-CPU systems. Extending it to handle multiple CPUs <span style="color:#f77729;"><b>efficiently</b></span> and reliably took a long time. To fill this gap, operating systems intended for single CPUs were initially extended to provide <span style="color:#f77729;"><b>minimal</b></span> support for a second CPU.

This is called <span style="color:#f77729;"><b>asymmetric</b></span> multiprocessing, in which each processor is assigned a <span style="color:#f77729;"><b>specific</b></span> task with the presence of a super processor that controls the system. This scheme defines a <span style="color:#f77729;"><b>master–slave</b></span> relationship. The master processor <span style="color:#f77729;"><b>schedules</b></span> and <span style="color:#f77729;"><b>allocates</b></span> work to the slave processors. The other processors either look to the master for instruction or have predefined tasks. 




# Appendix 2: Clustered System {#clustered-system}

Clustered systems (e.g: AWS) are multiple systems that are working together. They are usually:
1. Share <span style="color:#f77729;"><b>storage</b></span> via a storage-area-network
2. Provides <span style="color:#f77729;"><b>high availability</b></span> service which survives failures
3. Useful for <span style="color:#f77729;"><b>high performance computing</b></span>, require special applications that are written to utilize **parallelization**

There are two types of clustering:
1. <span style="color:#f77729;"><b>Asymmetric</b></span>: has one machine in hot-standby mode. There’s a master machine that runs the system, while the others are configured as slaves that receive tasks from the master. The master system does the basic input and output requests.
2. <span style="color:#f77729;"><b>Symmetric</b></span>: multiple machines (nodes) running applications, they are monitoring one another, requires complex algorithms to maintain data integrity. 


[^5]:
    Interrupt-driven I/O is fine for moving small amounts of data but can produce high overhead when used for bulk data movement such as disk I/O. To solve this problem, direct memory access (DMA) is used. After setting up buffers, pointers, and counters for the I/O device, the device controller transfers an entire block of data directly to or from its own buffer storage to memory, with no intervention by the CPU.

[^6]:
     The CPU cache is a hardware cache: performing optimization that’s unrelated to the functionality of the software. It handles each and every access between CPU cache and main memory as well. They need to work fast, too fast to be under software control, and _are entirely built into the hardware._

