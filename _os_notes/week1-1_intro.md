---
title: Introduction to OS
permalink: /os_notes/week1_intro
key: os-notes-week1_intro
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


An operating system  (OS) is a program that **manages computer hardware**. 
{:.info}

The figure below shows the hardware components of a common general purpose computer.  There are many user programs that are running in a computer, and the OS acts as an intermediary application that enables many user programs to share the same set of hardware, such as the mouse, printer, keyboard, display monitor, etc. 


<img src="/50005/assets/images/week1/1.png"  class="center_seventy"/>


## The Operating System {#the-operating-system}

An operating system is a **special program** that acts as an intermediary between users of the computer and the computer hardware. 

The goal of an operating system is such that we have a **dedicated program** to fulfil the following essential roles:
1. **Resource allocator and coordinator**: controls hardware and input/output requests, manage conflicting requests, manage interrupts
2. **Controls program execution**:
    * Storage hierarchy manager
    * Process manager 
3. Limits program execution and ensure **security**: 
    * Preventing illegal access to the hardware or improper usage of the hardware

Once we have an _operating system_, it makes things easier for users to **use** a program or **write** another program for other purposes within a **computer system.**

There are a lot of things that make up an operating system, but they are generally divided into three categories:

1. **The Kernel**
2. **System programs**
3. **User programs**

Definition and role of **<span style="color:#f77729;"><b>operating system kernel</b></span>** can be found in another section below, and it is the only program with <span style="color:#f7007f;"><b>full</b></span> privileges, i.e: absolute access to control all the hardware in the computer system. 

Both system programs and user programs run in <span style="color:#f7007f;"><b>user mode</b></span>, with limited privileges, i.e: any of these programs have to send a _request_ to the kernel each time they require access to the I/O or hardware devices. 


## The Computer System {#the-computer-system}

A computer system can be roughly divided into **four** components: the hardware, the operating system, the application programs, and the users as shown in the figure below. 


<img src="/50005/assets/images/week1/2.png"  class="center_seventy"/>

The operating system is part of the computer system and is analogous to a **government**.
{:.info}

The OS provides an _<span style="color:#f77729;"><b>environment</b></span>_ such that user programs such as the text editor, web browser, compiler, database system, music player, video editor, etc can do useful work. Since each user program runs in a <span style="color:#f77729;"><b>virtual machine</b></span> (i.e: it is written in a manner that the _entire_ machine belongs to itself), there has to be some sort of _manager_ program that **has higher privileges** and **oversees** all programs that live on the RAM and reside on disk, as well as managing the **memory hierarchy**. 

This special program is part of the operating system called the **<span style="color:#f7007f;"><b>kernel</b></span>**. 
{:.info}

## The Memory Hierarchy  {#the-memory-hierarchy}

The storage structure in a typical computer system is made of **registers**, **caches**, **main** **memory**, and  **non-volatile secondary storage** such as magnetic disk. The wide variety of storage systems can be arranged in terms of hierarchy according to **speed and cost** (increasing speed and increasing cost from bottom up):

<img src="/50005/assets/images/week1/3.png"  class="center_seventy"/>

* The CPU can load instructions only from memory, so any programs to run must be stored there. 
* General-purpose computers run most of their programs from rewritable memory, called main memory (RAM). 
* At each CPU clock cycle, instructions are fetched from the main memory to the CPU. 

### The RAM
Ideally, we want the programs and data to reside in main memory (also known as physical memory, or RAM) <span style="color:#f7007f;"><b>permanently</b></span>. This arrangement usually is **not possible** for two reasons:
1. Main memory is usually too small to store all needed programs and data permanently.
2. Main memory is a <span style="color:#f7007f;"><b>volatile</b></span> storage device that loses its contents when power is turned off or otherwise lost.

Recall that the memory unit sees only a stream of memory addresses; it does not know how they are generated (by the instruction counter, indexing, indirection, literal addresses, or some other means) or what they are for (instructions or data).
{:.info}

### Cache
Cache devices are typically used to <span style="color:#f77729;"><b>speed</b></span> up the performance of the computer. They are  storing (typically) a few of the most recently used instruction pages. Cache devices are wired directly to the CPU so that the  CPU has direct access to it (unlike secondary storages), just like how the CPU can directly access the RAM. 

You may think of it as a more-expensive, smaller-but-faster RAM. 
{:.info}

Because caches have limited size, cache management is an important design problem. The **kernel** dictates details pertaining to cache management, such as which supported <span style="color:#f77729;"><b>cache replacement policies</b></span> should be used for the system. 

