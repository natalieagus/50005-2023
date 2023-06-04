---
title: Processes and Threads
permalink: /os_notes/week3_intro
key: os-notes-week3_intro
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

## Process vs Program {#the-concept-of-process-vs-program}

→ A <span style="color:#f77729;"><b>process</b></span> is formally defined as a program in execution. Process is an active, <span style="color:#f7007f;"><b>dynamic</b></span> entity -- i.e: it changes state overtime during execution, while a program is a passive, <span style="color:#f7007f;"><b>static</b></span> entity.
{:.warning}

- <span style="color:#f7007f;"><b>A program is not a process</b></span>.
- A process is much more than just a program code (text or instruction section).

### A Process Context

A single process includes all of the following information:

1. The <span style="color:#f77729;"><b>text</b></span> section (code or instructions)
2. Value of the Program Counter (<span style="color:#f77729;"><b>PC</b></span>)
3. Contents of the processor’s <span style="color:#f77729;"><b>registers</b></span>
4. Dedicated[^1] <span style="color:#f77729;"><b>address space</b></span> (block of location) in memory
5. <span style="color:#f77729;"><b>Stack</b></span> (temporary data such as function parameters, return address, and local variables, grows downwards),
6. <span style="color:#f77729;"><b>Data</b></span> (allocated memory during compile time such as global and static variables that have predefined values)
7. <span style="color:#f77729;"><b>Heap</b></span> (dynamically allocated memory -- typically by calling `malloc `in C -- during process runtime, grows upwards)[^2]
   These information are also known as a process <span style="color:#f7007f;"><b>state</b></span>, or a process <span style="color:#f7007f;"><b>context</b></span>:

The same program can be run `n` times to create `n` processes simultaneously.

- For example, separate tabs on some web browsers are created as <span style="color:#f77729;"><b>separate processes</b></span>.
- The <span style="color:#f77729;"><b>program</b></span> for all tabs are the same: which is the part of the web browser code itself.

→ A program becomes a process when an <span style="color:#f77729;"><b>executable</b></span> (binary) file is <span style="color:#f77729;"><b>loaded</b></span> into memory (either by double clicking them or executing them from the command line)
{:.warning}

### Concurrency and Protection

→ A process couples <span style="color:#f7007f;"><b>two abstractions</b></span>: <span style="color:#f7007f;"><b>concurrency</b></span> and <span style="color:#f7007f;"><b>protection</b></span>.
{:.warning}

Each process runs in a different address space and sees itself as running in a virtual machine -- unaware of the presence of other processes in the machine. Multiple processes execution in a single machine is <span style="color:#f7007f;"><b>concurrent</b></span>, managed by the <span style="color:#f7007f;"><b>kernel scheduler</b></span>.

## Piggybacking

From this point onwards, we are going to refer simply to the scheduler as the kernel. Remember this is just a running code (service routine) with kernel privileges whose task is to manage processes. Scheduler in itself, is part of the kernel, and is <span style="color:#f7007f;"><b>not</b></span> a process.

Recall how the system calls <span style="color:#f7007f;"><b>piggyback</b></span> the currently running user-process which mode has been changed to kernel mode. Likewise, the scheduler is just a set of instructions, part of the kernel realm whose job is to <span style="color:#f77729;"><b>manage</b></span> other processes. It is inevitable for the scheduler to be executed as well upon invoking certain system calls such as `read` or get `stdin` input which requires some <span style="color:#f7007f;"><b>waiting</b></span> time.

# Process Scheduling States {#process-scheduling-state}

As a process executes, it changes its <span style="color:#f7007f;"><b>scheduling state</b></span> which reflects the <span style="color:#f7007f;"><b>current</b></span> activity of the process.
{:.warning}

In general these states are:

1. <span style="color:#f77729;"><b>New</b></span>: The process is being created.
2. <span style="color:#f77729;"><b>Running</b></span>: Instructions are being executed.
3. <span style="color:#f77729;"><b>Waiting</b></span>: The process is waiting for some event to occur (such as an I/O completion or reception of a signal).
4. <span style="color:#f77729;"><b>Ready</b></span>: The process is waiting to be assigned to a processor
5. <span style="color:#f77729;"><b>Terminated</b></span>: The process has finished execution.

The figure below shows the scheduling state transition diagram of a typical process:
<img src="/50005/assets/images/week3/1.png"  class="center_seventy"/>

## Process Table and Process Control Block {#process-control-block}

The system-wide process table is <span style="color:#f77729;"><b>data structure</b></span> maintained by the Kernel to facilitate **context switching** and **scheduling**. Each process metadata is <span style="color:#f77729;"><b>stored</b></span> by the Kernel in a particular data structure called the process control block (PCB). A process table is made up of an array of PCBs, containing information about of current processes[^3] in the system.
{:.warning}

> Also called a <span style="color:#f77729;"><b>task control block</b></span> in some textbooks.

The PCB contains many pieces of information associated with a specific process. These information are up<span style="color:#f77729;"><b></b></span>dated each time when a process is <span style="color:#f77729;"><b>interrupted</b></span>:

1. Process <span style="color:#f77729;"><b>state</b></span>: any of the state of the process -- new, ready, running, waiting, terminated
2. <span style="color:#f77729;"><b>Program counter</b></span>: the address of the _next instruction_ for this process
3. CPU <span style="color:#f77729;"><b>registers</b></span>: the contents of the registers in the CPU when an interrupt occurs, including stack pointer, exception pointer, stack base, linkage pointer, etc. These contents are saved each time to allow the process to be continued correctly afterward.
4. <span style="color:#f77729;"><b>Scheduling </b></span> information: access priority, pointers to scheduling queues, and any other scheduling parameters
5. <span style="color:#f77729;"><b>Memory-management</b></span> information: page tables, MMU-related information, memory limits
6. <span style="color:#f77729;"><b>Accounting</b></span> information: amount of CPU and real time used, time limits, account numbers, process id (<span style="color:#f77729;"><b>pid</b></span>)
7. <span style="color:#f77729;"><b>I/O status</b></span> information: the list of I/O devices allocated to the process, a list of open files

### Linux task_struct

In Linux system, the PCB is created in C using a data structure called `task_struct`. The diagram below[^4] illustrates some of the contents in the structure:
<img src="/50005/assets/images/week3/2.png"  class="center_fifty"/>

Do not memorize the above, it's just for illustration purposes only. The task_struct is a relatively large data structure, at around 1.7 kilobytes on a 32-bit machine.
{:.error}

Within the Linux kernel, all active processes are represented using a <span style="color:#f77729;"><b>doubly linked list</b></span> of `task_struct.` The kernel maintains a `current_pointer` to the process that's currently <span style="color:#f77729;"><b>running</b></span> in the CPU.

<img src="/50005/assets/images/week3/3.png"  class="center_fourty"/>

### Context Switching

When a CPU switches execution between one process to another, the Kernel has to <span style="color:#f77729;"><b>store</b></span> all of the process states onto its corresponding PCB, and <span style="color:#f77729;"><b>load</b></span> the new process’ information from its PCB before resuming them as shown below, (_image screenshot from SGG book_):

<img src="/50005/assets/images/week3/4.png"  class="center_fifty"/>

## Rapid Context Switching and Timesharing {#rapid-context-switching-and-timesharing}

<span style="color:#f77729;"><b>Context switch</b></span> definition: the mechanism of <span style="color:#f77729;"><b>saving</b></span> the states of the current process and <span style="color:#f77729;"><b>restoring</b></span> (loading) the state of a different process when switching the CPU to execute another process.
{:.warning}

### Timesharing Support

Timesharing requires <span style="color:#f77729;"><b>interactivity</b></span>, and this is done by performing rapid context switching between execution of multiple programs in the computer system.

When an interrupt occurs, the kernel needs to save the current context of the process running on the CPU so that it can restore that context when its processing is done, essentially <span style="color:#f77729;"><b>suspending</b></span> the process and then resuming it at the later time.

> The suspended process context is stored in the <span style="color:#f77729;"><b>PCB</b></span> of that process.

#### Benefits of Context Switching

Rapid Context switching is beneficial because it gives the <span style="color:#f77729;"><b>illusion</b></span> of concurrency in uniprocessor system.

- Improve system <span style="color:#f77729;"><b>responsiveness</b></span> and intractability, ultimately allowing timesharing (users can interact with each program when it is running).
- To support <span style="color:#f77729;"><b>multiprogramming</b></span>, we need to <span style="color:#f77729;"><b>optimise</b></span> CPU usage. We cannot just let one single program to run all the time, especially when that program <span style="color:#f77729;"><b>blocks</b></span> execution when waiting for I/O (idles, have nothing important to do).

### Drawbacks

Context-switch time is pure <span style="color:#f7007f;"><b>overhead</b></span>, because the system <span style="color:#f7007f;"><b>does no useful work</b></span> while switching. To minimise downtime due to overhead, context-switch times are highly dependent on hardware support -- some hardware supports rapid context switching by having a <span style="color:#f7007f;"><b>dedicated</b></span> unit for that (effectively bypassing the CPU).

## Mode Switch Versus Context Switch {#mode-switch-versus-context-switch}

Mode switch and Context switch -- although similar in name; they are two separate concepts.
{:.warning}

### Mode switch

- The <span style="color:#f77729;"><b>privilege</b></span> of a process changes
- Simply escalates privilege from user mode to kernel mode to access kernel services.
- Done by either: hardware interrupts, system calls (traps, software interrupt), exception, or reset
- Mode switch <span style="color:#f7007f;"><b>may not always lead to context switch</b></span>. Depending on implementation, Kernel code decides whether or not it is necessary.

### Context switch

- Requires <span style="color:#f77729;"><b>both</b></span> saving (all) states of the old process and loading (all) states of the new process to resume execution
- Can be caused either by timed interrupt or system call that leads to a `yield()`, e.g: when waiting for something.

Think about scenarios that requires <span style="color:#f77729;"><b>mode switch</b></span> but <span style="color:#f7007f;"><b>not</b></span> context switch.
{:.error}

## Process Scheduling {#process-scheduling}

### Motivation

The objective of <span style="color:#f77729;"><b>multiprogramming</b></span> is to have some process running at all times, to maximize CPU utilization.

The objective of <span style="color:#f77729;"><b>time sharing</b></span> is to switch the CPU among processes so frequently that users can interact with each program while it is running.

To meet <span style="color:#f77729;"><b>both</b></span> objectives, the process scheduler selects an available process (possibly from a set of several available processes) for program execution on the CPU.

- For a single-processor system, there will never be more than one actual running process at any instant.
- If there are more processes, the rest will have to wait in a queue until the CPU is free and can be rescheduled.

For Linux scheduler, see the man page [here](https://man7.org/linux/man-pages/man7/sched.7.html). We can set scheduling policies: `SCHED_RR`, `SCHED_FIFO`, etc. We can also set <span style="color:#f77729;"><b>priority</b></span> value and `nice` value of a process (the latter used to control the priority value of a process in user space[^5].

### Process Scheduling Queues

There are three <span style="color:#f77729;"><b>queues</b></span> that are maintained by the process <span style="color:#f77729;"><b>scheduler</b></span>:

1. <span style="color:#f7007f;"><b>Job</b></span> queue – set of all processes in the system (can be in both main memory and swap space of disk)
2. <span style="color:#f7007f;"><b>Ready</b></span> queue – set of all processes residing in main memory, ready and waiting to execute (queueing for CPU)
3. <span style="color:#f7007f;"><b>Device</b></span> queues – set of processes waiting for an I/O device (one queue for each device)

Each queue contains the <span style="color:#f7007f;"><b>pointer</b></span> to the corresponding PCBs that are waiting for the <span style="color:#f7007f;"><b>resource</b></span> (CPU, or I/O).

#### Example

The diagram below shows a system with ONE ready queue, and FOUR device queues (_image screenshot from SGG book_):

<img src="/50005/assets/images/week3/5.png"  class="center_seventy"/>

#### Queueing Diagram

A common representation of process scheduling is using a queueing diagram as shown below:

<img src="/50005/assets/images/week3/6.png"  class="center_seventy"/>

Legends:

- <span style="color:#f77729;"><b>Rectangular</b></span> boxes represent queues,
- <span style="color:#f77729;"><b>Circles</b></span> represent resources serving the queue
- All types of queue: job queue, ready queue and a set of device queues (I/O queue, I/O Req, time exp, fork queue, and wait irq)

A new process is initially put in the <span style="color:#f77729;"><b>ready</b></span> queue. It waits there until it is selected for execution, or dispatched. Once the process is allocated the CPU, a few things <span style="color:#f77729;"><b>might</b></span> happen afterwards (that causes the process to leave the CPU):

- If the process issue an I/O request, it will be placed onto the <span style="color:#f77729;"><b>I/O queue</b></span>
- If the process `forks` (<span style="color:#f77729;"><b>create</b></span> new process), it may queue (wait) until the child process finishes (terminated)
- The process could be forcily removed from the CPU (e.g: because of an interrupt), and be put back in the <span style="color:#f77729;"><b>ready</b></span> queue.

### Long and Short Term Scheduler

Scheduler is typically divided into two parts: long term and short term. They manage each queue accordingly as shown:

<img src="/50005/assets/images/week3/7.png"  class="center_seventy"/>

[^1]: Because each process is isolated from one another and runs in different address space (forming virtual machines)
[^2]: Heap and stack grows in the opposite direction so that it maximises the space that both can have and minimises the chances of overlapping, since we do not know how much they can dynamically grow during runtime. If the heap / stack grows too much during runtime, we are faced with stack/heap overflow error. If there’s a heap overflow, `malloc` will return a `NULL` pointer.
[^3]: You can list all processes that are currently running in your UNIX-based system by typing `ps aux` in the command line.
[^4]: Image referenced from IBM RedBooks – Linux Performance and Tuning Guidelines.
[^5]: The relation between nice value and priority is: Priority_value = Nice_value + 20
