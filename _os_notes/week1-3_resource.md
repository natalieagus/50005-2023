---
title: Resource Allocator and Coordinator
permalink: /os_notes/week1_resource
key: os-notes-week1_resource
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

The kernel <span style="color:#f77729;"><b>controls</b></span> and <span style="color:#f77729;"><b>coordinates</b></span> the use of hardware and I/O devices among various user applications. Examples of I/O devices include mouse, keyboard, graphics card, disk, network cards, among many others.

# Interrupt-Driven I/O Operations

Interrupt-Driven I/O operations allow the CPU to efficiently handle interrupts without having to waste resources; waiting for asynchronous interrupts. This notes explains how interrupt-driven I/O operations work in a nutshell. There are <span style="color:#f7007f;"><b>two kinds of interrupts</b></span>:

1. <span style="color:#f7007f;"><b>Hardware Interrupt</b></span>: input from external devices activates the interrupt-request-line, thus pausing the current execution of user programs.
2. <span style="color:#f7007f;"><b>Software Interrupt</b></span>: a software generated interrupt that is invoked from the instruction itself because the current execution of user program needs to access Kernel services.

# Hardware Interrupt

The CPU has an <span style="color:#f77729;"><b>interrupt-request line</b></span> that is sensed by its control unit before each instruction execution.

- Upon the presence of new **input**: from <span style="color:#f77729;"><b>external</b></span> events such as keyboard press, mouse click, mouse movement, incoming fax, or **completion of previous I/O requests** made by the drivers on <span style="color:#f77729;"><b>behalf</b></span> of user processes, the device controllers will invoke an **interrupt** request by setting the **bit** on this <span style="color:#f77729;"><b>interrupt-request line</b></span>.

Remember that this is a <span style="color:#f7007f;"><b>hardware interrupt</b></span>: an interrupt that is caused by setting the interrupt-request line.
{:.error}

This forces the CPU to **transfer** control to the **interrupt handler**. This switch the currently running user-program to enter the <span style="color:#f77729;"><b>kernel mode</b></span>. The interrupt handler will do the following routine:

1. **<span style="color:#f7007f;"><b>Save</b></span> the register states** first (the interrupted program instruction) into the process table
2. And then transferring control to the appropriate interrupt service routine — depending on the device controller that made the request.

## Vectored Interrupt System

Interrupt-driven system may use a <span style="color:#f77729;"><b>vectored interrupt system</b></span>: the interrupt signal that **INCLUDES** the identity of the device sending the interrupt signal, hence allowing the kernel to know exactly which interrupt service routine to execute[^2]. This is more <span style="color:#f7007f;"><b>complex</b></span> to implement, but more <span style="color:#f77729;"><b>useful</b></span> when there are sparse I/O requests.

This interrupt mechanism accepts an **address**, which is usually one of a small set of numbers for an offset into a table called the **interrupt vector.** This table holds the addresses of routines prepared to process specific interrupts.
{:.info}

## Polled Interrupt System

An alternative is to use a <span style="color:#f77729;"><b>polled interrupt system</b></span>:

- The interrupted program enters a general interrupt polling routine <span style="color:#f7007f;"><b>protocol</b></span>, where CPU <span style="color:#f7007f;"><b>scans</b></span> (polls) devices to determine which device made a service request.
- Unlike vectored interrupt, there’s no such _interrupt_ signal that includes the identity of the device sending the interrupt signal.
- In the polled system, the kernel must send a signal out to each controller to **determine** if any device made a service request <span style="color:#f7007f;"><b>periodically, or at any fixed interval</b></span>.

This is simpler to implement, but more time-wasting if there’s sparse I/O requests.
Does Linux implement a Polled interrupt or Vectored interrupt system? See [here](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html) for clues.
{:.info}

## Multiple Interrupts

If there’s more than one I/O interrupt requests from multiple devices, the Kernel may decide which interrupt requests to service first. When the service is done, the Kernel scheduler may **choose** to resume the user program that was interrupted.

## Raw Device Polling

It takes time to perform a **<span style="color:#f7007f;"><b>context switch</b></span>** to handle interrupt requests. In some specifically dedicated servers, raw polling may be faster. The CPU <span style="color:#f7007f;"><b>periodically</b></span> checks each device for requests. If there's no request, then it will return to resume the user programs. Obviously, this method is <span style="color:#f77729;"><b>not suitable</b></span> in the case of sporadic I/O usage, or in general-purpose CPUs since at least a fixed bulk of the CPU load will always be dedicated to perform routine polling. For instance, there's no need to poll for anything periodically when a user is watching a video.

Modern computer systems are **<span style="color:#f7007f;"><b>interrupt-driven</b></span>**, meaning that it <span style="color:#f7007f;"><b>does not wait</b></span> until the input from the requested external device is ready. This way, the computer may <span style="color:#f77729;"><b>efficiently</b></span> fetch I/O data **only when the devices are `ready`{:.error},** and will not waste CPU cycles to "poll" whether there are any I/O requests or not.

## Hardware Interrupt Timeline

The figure below summarises the **interrupt-driven** procedure of **asynchronous I/O handling** during **<span style="color:#f7007f;"><b>hardware interrupt</b></span>**:

<img src="/50005/assets/images/week1/9.png"  class="center_seventy"/>

Notes:

1. **I/O Devices**: External I/O Device, e.g: printer, mouse, keyboard. Runs asynchronously, capable of being "smart" and run its own instructions.
2. **Device Controller**: Attached to motherboard. Runs asynchronously, may run simple instructions. Has its own buffers, registers, and simple instruction interpreter. Usually transfer data to and fro the I/O device via standard protocols such as USB A/C, HDMI, DP, etc.
3. **Disk Controller**: Same as device controller, just that it's specific to control Disks

### Receiving asynchronous Input

From the figure above, you may assume that at first the CPU is busy executing user process instructions. At the same time, the device is _idling_.

Upon the presence of external events, e.g: mouse `CLICK()`, the device <strong>records</strong> the input. This triggers <span style="color:#f7007f;"><b>step 1</b></span>: The I/O device sends data to the device controller, and <strong>transfers </strong>the data from the<strong> device buffer </strong>to the<strong> local buffer of the device controller.</strong>

When I/O transfer is complete, this triggers <span style="color:#f7007f;"><b>step 2</b></span>: the device controller makes an <strong>interrupt request</strong> to signal that a <em>transfer is done (and data needs to be fetched)</em>. The simplest way for the device controller to raise an interrupt is by asserting a signal on the interrupt request line (this is why we define I/O interrupts as hardware generated)

The interrupt request line is sensed by the CPU at the beginning of each instruction execution, and when there’s an interrupt, the execution of the <strong>current user program is interrupted.</strong> Its states are <span style="color:#f7007f;"><b>saved</b></span>[^3] by the entry-point of the interrupt handler. Then, this handler determines the <span style="color:#f7007f;"><b>source</b></span> of the interrupt (be it via Vectored or Polling interrupt) and performs the necessary processing. This triggers <span style="color:#f7007f;"><b>step 3</b></span>: the CPU executes the proper I/O service routine to transfer the data <span style="color:#f77729;"><b>from</b></span> the local device controller buffer <span style="color:#f77729;"><b>to</b></span> the physical memory.

After the I/O request is serviced, the handler:

1. <span style="color:#f77729;"><b>Clears</b></span> the interrupt request line,
2. <span style="color:#f77729;"><b>Performs</b></span> a state restore, and executes a **_return from interrupt_** instruction (or `JMP(XP)` as you know it from 50.002).

### Consuming the Input

Two things may happen from here after we have stored the new input to the RAM:

1. If there’s no application that’s currently waiting for this input, then it might be temporarily stored somewhere in kernel space first.
2. If there is **any application** that is waiting (blocked, like Python's `input()`) for this input (e.g: mouse click), that process will be labelled as <span style="color:#f77729;"><b>ready</b></span>. For example, if the application is blocked upon waiting for this new input, then the system call returns. **We will learn more about this in Week 3.**

One thing should be crystal clear: in an interrupt-driven system, upon the presence of new input, a **hardware interrupt** occurs, which invokes the interrupt handler and then the interrupt service routine to service it. <span style="color:#f7007f;"><b>It does not matter whether any process is currently waiting for it or not</b></span>. If there’s a process that’s waiting for it, then it will be scheduled to resume execution since its system call will **return** (if the I/O request is blocking).
{:.info}

# Software Interrupt (Trap) via System Call

Firstly, this image says it all.

<img src="/50005/assets/images/week1/10.png"  class="center_fifty"/>

Sometimes user processes are <span style="color:#f77729;"><b>blocked</b></span> from execution because it requires inputs from IO devices, and it may **not be scheduled** until the presence of the required input arrives. For example, this is what happens if you wait for user input in Python:

```python
inp = input("Enter your name:")
print(inp)
```

Running it will just cause your program to be stuck in the console, _why_:

```console
bash-3.2$ python3 test.py
Enter your name:
```

User processes may <span style="color:#f7007f;"><b>trap</b></span> themselves e.g: by executing an <span style="color:#f7007f;"><b>illegal</b></span> operation (`ILLOP`) implemented as a system call, along with some value in designated registers to <span style="color:#f77729;"><b>indicate</b></span> which system call this program tries to make.

> The details on _which register_, or _what value should be put_ into these registers depends on the architecture of your CPU. <span style="color:#f77729;"><b>This detail is not part of our syllabus</b></span>.

Traps are **software generated interrupts**, that is some special instructions that will <span style="color:#f77729;"><b>transfer</b></span> the mode of the program to the Kernel Mode.
{:.info}

The CPU is forced to go to a special handler that does a state save and then execute (may not be immediate!) on the proper interrupt service routine to handle the <span style="color:#f7007f;"><b>request</b></span> (e.g: fetch user input in the python example above) in kernel mode. Software interrupts generally have <span style="color:#f77729;"><b>low priority</b></span>, as they are not as urgent as devices with limited buffering space.

<img src="/50005/assets/images/week1/11.png"  class="center_seventy"/>

During the time between system call request until system call return, the program execution is <span style="color:#f7007f;"><b>paused</b></span>. Examples of system calls are: `chmod(), chdir(), print()`. More Linux system calls can be found [here](http://man7.org/linux/man-pages/man2/syscalls.2.html).

### Combining Hardware Interrupt and Trap

Consider another scenario where you want to open a **very large file** from disk. It takes some time to <span style="color:#f7007f;"><b>load</b></span> (simply transfer your data from disk to the disk controller), and your CPU can proceed to do other tasks in the meantime. Here's a simplified timeline:

<img src="/50005/assets/images/week1/12.png"  class="cenetr_full"/>

Imagine that at first, the CPU is busy executing process instructions in user mode. At the same time, the device is idling.

1. The process requests for Kernel services (e.g: load data asynchronously) by making a <span style="color:#f7007f;"><b>system call</b></span>.
   - The **context** of the process are saved by the trap handler
   - Then, the appropriate system call service routine is called. Here, they may require to load appropriate **device drivers** so that the CPU may communicate with the device controller.
2. The **device controller** then makes the instructed I/O request to the device itself on behalf of the CPU, **e.g: a disk,** as instructed.
   - Meanwhile, the service handler returns and may resume the calling process as illustrated.

The I/O device then proceeds on responding to the request and **transfers** the data from the **device** to the **local buffer** of the device controller.
{:.info}

When I/O transfer is <span style="color:#f7007f;"><b>complete</b></span>, the device controller makes an **<span style="color:#f7007f;"><b>hardware interrupt</b></span> request** to signal that the transfer is done (and data needs to be fetched). The CPU may respond to it by saving the states of the currently interrupted process, handle the interrupt, and resume the execution of the interrupted process.

**Note**:

1. SVC <span style="color:#f77729;"><b>delay</b></span> and IRQ <span style="color:#f77729;"><b>delay</b></span>: time elapsed between when the request is invoked until when the request is first executed by the CPU.
2. Before the user program is resumed, its state must be <span style="color:#f77729;"><b>restored</b></span>. Saving of state during the switch between User to Kernel mode is implied although it is not drawn.

# Reentrancy

A reentrant kernel is the one which allows <span style="color:#f7007f;"><b>multiple</b></span> processes to be executing in the kernel mode at any given point of time, hopefully without causing any consistency problems among the kernel data structures. If the kernel is not re-entrant, a process can only be suspended <span style="color:#f77729;"><b>while it is in user mode</b></span>.

In a non-reentrant kernel: although a process could be suspended in kernel mode, that would still <span style="color:#f77729;"><b>block</b></span> kernel mode execution on <span style="color:#f77729;"><b>all other processes</b></span>.

For example, consider Process 1 that is <span style="color:#f7007f;"><b>voluntarily</b></span> paused (suspended) when it is in the middle of handling its `async_load` system call. It is <span style="color:#f77729;"><b>suspended in Kernel Mode</b></span> by `yielding` itself.

- In a reentrant kernel: Process 2 is currently executed; able to be handling its `print` system call as well.
- In a non-reentrant kernel: Process 2, although currently executed must **wait** for Process 1 to exit from the Kernel Mode if Process 2 wishes to execute its `print` system call.

In simpler Operating Systems, incoming hardware interrupts are typically <span style="color:#f7007f;"><b>disabled</b></span> while another interrupt (of same or higher priority) is being processed to prevent a lost interrupt i.e: when user states are currently being saved before the actual interrupt service routine began or various **<span style="color:#f7007f;"><b>reentrancy problems</b></span>**[^4].

# Preemption

A pre-emptive Kernel <span style="color:#f77729;"><b>allows the scheduler</b></span> to <span style="color:#f7007f;"><b>interrupt</b></span> processes in Kernel mode to execute the highest priority task that are ready to run, thus enabling kernel functions to be <span style="color:#f7007f;"><b>interrupted</b></span> just like regular user functions. The CPU will be assigned to perform other tasks, from which it later returns to finish its kernel tasks. In other words, the scheduler is permitted to <span style="color:#f7007f;"><b>forcibly</b></span> perform a context switch.

Likewise, in a non-preemptive kernel the scheduler is not capable of rescheduling a task while its CPU is executing in the kernel mode.
{:.info}

In the example of Process 1 and 2 above, assume a scenario whereby there's a periodic scheduler interrupt to check _which_ Process may resume next. Assume that Process 1 is in the middle of handling its `async_load` system call when the timer interrupts.

- In a non-preemptive Kernel: If Process 1 does not voluntarily `yield` while it is still in the middle of its system call, then Process 2 will not be able to forcibly interrupt Process 1.
- In a preemptive Kernel: When Process 2 is ready, and has a higher priority than Process 1, then the scheduler may **forcibly** suspend Process 1.

A kernel can be reentrant but not preemptive: That is if each process voluntarily `yield` after some time while in the Kernel Mode, thus allowing other processes to progress and enter Kernel Mode as well. However, a kernel <span style="color:#f7007f;"><b>should not</b></span> be preemptive and not reentrant (it doesn't make sense!).

**Fun fact:** Linux Kernel is reentrant and preemptive.
{:.info}

# Timed Interrupts {#timed-interrupts}

We must ensure that the kernel, as a <span style="color:#f7007f;"><b>resource allocator</b></span> maintains <span style="color:#f77729;"><b>control</b></span> over the CPU. We cannot allow a user program to get stuck in an infinite loop and never return control to the OS. We also cannot trust a user program to voluntarily return control to the OS. To ensure that no user program can occupy a CPU for indefinitely, a computer system comes with a (<span style="color:#f77729;"><b>hardware</b></span>)-based timer. A timer can be set to invoke the hardware interrupt line so that a running user program may transfer control to the kernel after a specified period. Typically, a <span style="color:#f7007f;"><b>scheduler</b></span> will be invoked each time the timer interrupt occurs.

In the hardware, a timer is generally implemented by a **fixed-rate clock** and a counter. The kernel may set the starting value of the counter, just like how you implement a custom clock in your 1D 50.002 project, for instance, here's one simple steps describing how timed interrupts work:

1. Every time the sytem clock ticks, a counter with some starting value `N` is decremented.
2. When the counter reaches a certain value, the timer can be hardware triggered to set the <span style="color:#f77729;"><b>interrupt request line</b></span>

For instance, we can have a 10-bit counter with a 1-ms clock can be set to trigger interrupts at intervals anywhere from 1 ms to 1,024 ms, with precision of 1 ms. Let's say we decide that every 512 ms, timed interrupt will occur, then we can say that the size of a **quantum** (time slice) for each process is 512ms. We can give multiple quantums (quanta) for a user process.

When timed interrupt happens, this transfers control over to the interrupt handler, and the following routine is triggered:

- <span style="color:#f77729;"><b>Save</b></span> the current program's state
- Then call the <span style="color:#f77729;"><b>scheduler</b></span> to perform context switching
- The scheduler may then reset the counter before restoring the next process to be executed in the CPU. This ensures that a proper timed interrupt can occur in the future.
- Note that a scheduler may allocate <span style="color:#f77729;"><b>arbitrary</b></span> amount of time for a process to run, e.g: a process may be allocated a longer time slot than the other. We will learn more about process management in Week 3.

# Exceptions {#exceptions}

Exceptions are <span style="color:#f77729;"><b>software interrupts</b></span> that occur due to <span style="color:#f7007f;"><b>errors</b></span> in the instruction: such as division by zero, invalid memory accesses, or attempts to access kernel space illegally. This means that the CPU's hardware may be designed such that it checks for the presence of these <span style="color:#f7007f;"><b>serious</b></span> errors, and immediately invokes the appropriate handler via a pre-built <span style="color:#f7007f;"><b>event-vector table</b></span>. Below is an example of ARMv8-M event vector table. The table is typically implemented in the <span style="color:#f77729;"><b>lower</b></span> physical addresses in many architecture.

<img src="/50005/assets/images/week1/13.png"  class="center_seventy" title="Image taken from https://developer.arm.com/documentation/100701/0200/Exception-properties"/>

Each exception has an ID (associated number), a vector <span style="color:#f77729;"><b>address</b></span> that is the exception <span style="color:#f77729;"><b>entry</b></span> point in memory, and a <span style="color:#f77729;"><b>priority</b></span> level which determines the order in which multiple pending exceptions are handled. In ARMv8-M, the lower the priority number, the higher the priority level.

<span style="color:#f7007f;"><b>You don't have to memorise these, don't worry.</b></span>
{:.error}

[^2]: Since only a predefined number of interrupts is possible, **a table of pointers to interrupt routines** can be used instead to provide the necessary speed. The interrupt routine is called indirectly through the table, with no intermediate routine needed. Generally, the table of pointers is stored in low memory (the first hundred or so locations), and its values are determined at boot time. These locations hold the addresses of the interrupt service routines for the various devices. This array, or **interrupt vector**, of addresses is then indexed by a unique device number, given with the interrupt request, to provide the address of the interrupt service routine for the interrupting device. Operating systems as different as Windows and UNIX dispatch interrupts in this manner.
[^3]: Depending on the implementation of the handler, <strong>a <em>full </em>context switch (completely saving ALL register states, etc) may not be necessary</strong>. Some OS may implement interrupt handlers in low-level languages such that it knows exactly which registers to use and only need to save the states of these registers first before using them.
[^4]: A reentrant procedure can be interrupted in the middle of its execution and then safely be called again ("re-entered") before its previous invocations complete execution. There are several problems that must be carefully considered before declaring a procedure to be _reentrant_. All user procedures are reentrant, but it comes at a costly procedure to prevent lost data during interrupted execution.
