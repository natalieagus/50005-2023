---
title: The Kernel
permalink: /os_notes/week1_kernel
key: os-notes-week1_kernel
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


The one **program** that is running at all times in the computer is the kernel.
{:.info}

The Kernel is the **<span style="color:#f7007f;"><b>heart</b></span>** of an operating system.
* For example, Ubuntu OS is about 2.7GB, but its Kernel (Linux) size is only  about 70MB). 
* It operates on the <span style="color:#f7007f;"><b>physical space</b></span> — meaning that it has full knowledge of all <span style="color:#f7007f;"><b>physical</b></span> addresses instead of <span style="color:#f77729;"><b>virtual</b></span> addresses, and has the <span style="color:#f7007f;"><b>complete privilege</b></span> over all the hardware of the computer system. 
* The only way to access the kernel code is when a process runs in the Kernel mode, via very specific <span style="color:#f7007f;"><b>controlled entry points</b></span>. 
 
You have learned this before: for instance via `ILLOP`, `IRQ`, and `RESET`. 

# Kernel Mode
The kernel runs with special privileges, called the <span style="color:#f7007f;"><b>kernel mode</b></span>. It can do what normal user program cannot do:
1. Ultimate access and control to all hardware in the computer system (mouse, keyboard, display, network cards, disk, RAM, CPU, etc)
2. Know (and lives in) the physical address space and manages the memory hierarchy
3. Interrupt other user programs
4. Receive and manage I/O requests
5. Manage other user program locations on the RAM, the MMU, and schedule user program executions

In order for the **kernel** to have more _privileges_ than **other user mode programs**, the computer hardware has to support **dual mode operation.**
{:.info}

You have learned this before as well, i.e: `PC31` in the Beta CPU indicates whether the current instruction is run in the Kernel mode or the User mode. 



## Hardware Support for Dual Mode Operation {#dual-mode-operation}


<img src="/50005/assets/images/week1/4.png"  class="center_fifty"/>


The dual mode is possible <span style="color:#f7007f;"><b>iff</b></span> it is supported by the hardware. The kernel is also **uninterruptible** and this interruptible feature is also supported by the hardware.
{:.info}

### Differences between architectures
In 50.002, we have learned that the control logic unit **prevents** the PC to `JMP` to memory address with MSB bit of `1` (where the kernel program resides) when it is at memory address with MSB bit 0 (where user programs reside). 
* Also, the control logic unit does not *trap* the PC onto the handler when an interrupt signal is present if the PC is running in kernel mode (MSB of the PC is 1). 

In the Linux system, low memory is dedicated for the kernel and high memory is assigned for user processes. It is essentially the same as what we have learned before: the concept of having _dual_ mode and _hardware support_. 
* Its hardware prevents the PC from jumping _illegally_ (not via handlers) to a lower memory address (MSB = 0) when it was from a higher memory address (MSB = 1).

### The Big Picture

A general purpose CPU has at least dual mode operation that should supported by its hardware: 

1. **<span style="color:#f7007f;"><b>The Kernel mode</b></span>** (privileged) : the executing code has complete and unrestricted access to the underlying hardware. 
2. **<span style="color:#f77729;"><b>The User mode</b></span>** (unprivileged) : all user programs such as a web browser, word editor, etc and also system programs such as compiler, assembler, file explorer, etc. Runs on _virtual machine_

User programs have to perform **system calls** (supervisor call) when they require services from the kernel, such as access to the hardware or I/O devices. When they perform **system calls**, the user program changes its mode to **the kernel mode** and began executing the kernel instructions handling that call instead of their own program instructions. When the system call returns, the `PC` resumes the execution of the user program. 


# Booting  {#booting}

Booting is the process of starting up a computer. **It is usually hardware initiated** — (by the start button that users press) — meaning that users physically initiate simple hardwired procedures to kickstart the chain of events that loads the firmware (BIOS) and eventually the entire OS to the main memory to be executed by the CPU.  This process of loading basic software to help kickstart operation of a computer system after a hard reset or power on is called **bootstrapping**. 

Recall that programs (including the operating system kernel) must **load** into the main memory before it can be executed. However, at the instance when the start button is pressed, there’s no program that resides in the RAM yet and therefore nothing can be executed in the CPU. 

We require a software to load another software into the RAM — this results in a <span style="color:#f7007f;"><b>paradox</b></span>.
{:.info}

## The Booting Paradox
To solve this paradox, the bare minimum that should be done in the hardware level upon pressing of the start button is to load a _special_ program onto the main memory from a dedicated input unit: **a <span style="color:#f77729;"><b>read-only-memory</b></span> (ROM) that comes with a computer when it is produced and that cannot be erased.** This special program is generally known as **firmware or BIOS**[^1].

After the firmware is loaded onto the main memory through hardwired procedures, the CPU may execute it and **initialise all aspects of the system, such as:**
1. <span style="color:#f77729;"><b>Prepare</b></span> all attached devices in a state that is ready to be used by the OS
2. <span style="color:#f77729;"><b>Loads</b></span> other programs — which in turn loads more and more complex programs,
3. <span style="color:#f77729;"><b>Loads</b></span> the Kernel from disk 
4. When the system boots, the hardware starts in the **kernel mode**. After being loaded, the Kernel will perform the majority of system setups (driver init, memory management, interrupts, etc). Afterwards, the rest of the OS is loaded and then user processes are started in _<span style="color:#f77729;"><b>user mode</b></span>_. 


The figure below summarises the booting process:
<img src="/50005/assets/images/week1/5.png"  class="center_seventy"/>

Note that the figure is heavily simplified for illustration purposes only.

# Computer System I/O Operation {#computer-system-i-o-operation}
There are **two** types of hardware in the computer system that are capable of running instructions:
1. The <span style="color:#f7007f;"><b>CPU</b></span> (obviously!)
2. <span style="color:#f77729;"><b> I/O device controllers</b></span>

Each I/O device is managed by an autonomous hardware entity called the **<span style="color:#f77729;"><b>device controllers</b></span>** as shown in the figure below:

<img src="/50005/assets/images/week1/6.png"  class="center_seventy"/>

In other words, I/O devices and the CPU can execute **instructions** in parallel. They are independent of one another and are <span style="color:#f7007f;"><b>asynchronous</b></span>
{:.info}




## Device Drivers  {#device-drivers}

A system must have **device drivers installed** for each device type. This driver is a specific program to <span style="color:#f7007f;"><b>interpret</b></span> the behavior of each device type. We typically install/download _device drivers_ when we plug in **new** I/O units to our computers through the USB port. 

<img src="/50005/assets/images/week1/7.png"  class="center_seventy"/>


It provides a software **interface** to hardware devices so that the device controller is able to communicate with the OS or an application program. 
{:.info}

### Running Device Drivers

Some default device drivers are part of the kernel code, and may consist of interfaces that control one or more common devices that can be attached to a system, such as hard disks, GPUs, keyboards, mouses, monitors, and network interfaces. In other words, device drivers are modules that can be <span style="color:#f77729;"><b>plugged</b></span> into an OS to handle a particular device or category of similar devices. Many drivers run in kernel mode (therefore some requires _reboot upon installation_), but there are also drivers that can run in user mode.

Those drivers that run in user mode will be _<span style="color:#f77729;"><b>slower</b></span>_ in comparison, since <span style="color:#f77729;"><b>frequent switching</b></span> to kernel mode is required to access the **serial ports** of the device controller that’s connected to the external devices. However, if it was <span style="color:#f7007f;"><b>poorly</b></span> written, <span style="color:#f77729;"><b>it will not endanger the system </b></span>by, for example, accidentally overwriting the kernel memory. 

For drivers that run in kernel mode, <span style="color:#f7007f;"><b>vulnerabilities</b></span> in these drivers can pose a serious threat as they can allow an attacker to escalate privileges to the highest level and become highly <span style="color:#f7007f;"><b>persistent</b></span>. 

One should only install drivers from <span style="color:#f77729;"><b>trusted</b></span> vendors.
{:.error}


## Device Controllers {#device-controllers}

Device controllers are **<span style="color:#f7007f;"><b>electronic components</b></span>** inside a computer that are in charge of specific types of devices. Components that make up the **device controllers:**
1. <span style="color:#f77729;"><b>Registers</b></span> — contains instructions that can be read by an appropriate **device driver program at the CPU **
2. Local memory <span style="color:#f77729;"><b>buffer</b></span>  — contains instructions and  data that will be fetched by the CPU when executing the device driver program, and ultimately loaded onto the RAM. 
3. A <span style="color:#f77729;"><b>simple</b></span> program to _communicate_ with the device driver

I/O operation happens when there’s transfer of data between the local memory buffer of the device controller and the device itself.
{:.info}

The I/O operation simply means:
1. Output: <span style="color:#f77729;"><b>Move</b></span> data from the RAM to the device controller’s buffer, or 
2. Input: <span style="color:#f77729;"><b>From</b></span> the device controller’s buffer to the RAM. 

Since the device controller and our CPU are <span style="color:#f7007f;"><b>asynchronous</b></span> (can operate independently, in parallel), we need to devise a way to <span style="color:#f7007f;"><b>coordinate</b></span> between servicing I/O requests and executing other user programs. This <span style="color:#f77729;"><b>I/O handling</b></span> issue is handled by our OS Kernel (read along to understand *how*).


# Roles of an Operating System Kernel {#roles-of-an-operating-system-kernel}

There are several purposes of an operating system: as a <span style="color:#f77729;"><b>resource</b></span> allocator, <span style="color:#f77729;"><b>controls</b></span> program execution, and guarantees <span style="color:#f77729;"><b>security</b></span> in the computer system. The next few notes will touch on each of these topics.



[^1]:
    Firmware is not equivalent to BIOS, but unfortunately some resources and PC manufacturers might just use them interchangeably. Firmware  generally refers to software stored on the motherboard (of any devices like computers, routers, switches,etc), containing basic settings of the device at startup. Some firmwares are upgradable, while some are Read-Only. BIOS is a term generally used specifically to refer to computer’s motherboard firmware in older computers. Modern computers use other Firmwares such as UEFI, also stored on chips on the motherboard. Note that UEFI / BIOS don’t form the entirety of a motherboard’s firmware.
