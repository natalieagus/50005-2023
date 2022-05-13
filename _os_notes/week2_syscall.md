---
title: System Calls
permalink: /os_notes/week2_syscall
key: os-notes-week2_syscall
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


Besides using the GUI, we can access OS services using its programming interface (system calls), meaning that we <span style="color:#f77729;"><b>develop</b></span> our own programs that rely on OS services to utilise the computer system’s hardware and I/O devices. 

System calls are programming interfaces provided by the OS Kernel for users to access kernel services. Unlike I/O interrupts, system calls are software generated interrupts (trap instruction). 
{:.info}

When application programs make system calls, the execution of its original instruction is temporarily suspended and switches to the Kernel mode to execute the system call routine. System calls are one of the <span style="color:#f77729;"><b>controlled</b></span> entry points to the kernel (besides interrupts and reset), meaning that we cannot make our program such that our `PC` directly executes arbitrary parts of the kernel space. System calls are the only way for user-mode processes to change into kernel-mode processes via software instructions.

This is usually supported by :
1. <span style="color:#f77729;"><b>Hardware</b></span>,  e.g.: PC cannot perform `JMP` to code with raw RAM addresses starting with MSB of ‘1’
2. <span style="color:#f77729;"><b>Virtualisation</b></span>, user programs operate on virtual memory

System calls are mostly accessed by programs through <span style="color:#f7007f;"><b>APIs</b></span> (application program interface), although we can certainly make system calls directly in assembly. There are plenty of examples in the next few sections.  

A system call does <span style="color:#f7007f;"><b>NOT</b></span> generally require a <span style="color:#f7007f;"><b>full</b></span> context switch; instead, it is processed in the context of whichever process invoked it.

## Accessing System Calls {#extras-accessing-system-calls}

Take this section with a grain of salt. It is not part of the syllabus but it is handy for you to understand how a regular user process can access kernel code via system calls. 
{:.warning}

All processes running in a computer must be able to make system calls. As a result, at the minimum the <span style="color:#f7007f;"><b>entry</b></span> points into the kernel have to be <span style="color:#f77729;"><b>mapped</b></span> into the current address space at all times.

To provide you with a context, let’s see a typical memory layout[^2] of a UNIX process (actual implementation may vary, e.g. Kernel is at low address space instead):
<img src="/50005/assets/images/week2/1.png"  class="center_full"/>


The virtual address space of a process is typically divided into two parts: <span style="color:#f77729;"><b>kernel</b></span> part in the higher address and <span style="color:#f77729;"><b>user</b></span> part in the lower address (or vice versa, depending on the Kernel implementation).

The kernel mapping part exists primarily for the<span style="color:#f77729;"><b> kernel-related purposes</b></span>, not user processes. Processes running in user mode don’t have access to the kernel’s address space (with different MSB), at all. In user mode, there is a <span style="color:#f77729;"><b>single</b></span> mapping for the kernel, shared across all processes, e.g: fixed address between `0xffffffff` to `0xC0000000` as illustrated above. 
* When a kernel-side page mapping changes, that change is reflected everywhere.

### Kernel Space

The kernel is divided into two spaces: <span style="color:#f77729;"><b>logical</b></span> and <span style="color:#f77729;"><b>virtual</b></span>, often called `lowmem` and `vmalloc` respectively.

In `lowmem`, it often uses a <span style="color:#f77729;"><b>one-to-one</b></span> mapping between virtual and physical addresses (its called logical mapping). That means virtual address `X` is mapped to physical address `X+C` (where `C` is some constant if any). This mapping is built during <span style="color:#f77729;"><b>boot</b></span>, and is <span style="color:#f77729;"><b>never</b></span> changed.

The kernel virtual address area (`vmalloc`) is used for <span style="color:#f77729;"><b>non-contiguous</b></span> physical memory location, so that it is <span style="color:#f77729;"><b>easier</b></span> to allocate them. 
* This allocation of process memory is <span style="color:#f77729;"><b>dynamic</b></span> and on demand. 
* On each allocation, a series of locations of physical pages are found for the corresponding kernel virtual address range, and the pagetable is <span style="color:#f77729;"><b>modified</b></span> to create the mapping.
* If this is done, it might be <span style="color:#f77729;"><b>unsuitable</b></span> for DMA (Direct Memory Access).


# System Calls via API {#system-calls-through-api}

One of the most common ways to make system calls is through an API (Application Programming Interface). We can write a program in a particular language and conveniently perform several system calls through the provided APIs supported by the chosen language as needed.

About API: API is an interface that provides a way to interact with the underlying library[^3] that makes the system calls, often named the same as the system calls they invoke. It specifies: 



* a set of functions that are available to an application programmer
* the parameters that are passed to each function and 
* the return values the programmer can expect

3 most common APIs available:



1. Win32 API for Windows systems -- written in C++
2. POSIX API for POSIX-based system (all versions of UNIX) -- mostly written in C. You can find the functions supported by the API here  [https://pubs.opengroup.org/onlinepubs/9699919799/](https://pubs.opengroup.org/onlinepubs/9699919799/)
3. Java API for programs running on Java Virtual Machine (JVM)

Behind the scenes, the functions that make up an API invoke the actual system calls on behalf of the application programmer.



Benefits of APIs:



* It adds another layer of abstraction hence simplifies the process of application development[^4]


* Supports program portability[^5]
_Note: Making the system call directly in the application code is possible, but more complicated and may require embedded assembly code to be used (in C and C++) as well as knowledge of the low-level binary interface for the system call operation, which may be subject to change over time and thus not be part of the application binary interface; the API is meant to abstract this away. _

To see more examples about _direct _system calls, you can find out more about Linux[^6] system calls in [http://man7.org/linux/man-pages/man2/syscalls.2.html](http://man7.org/linux/man-pages/man2/syscalls.2.html) 

Example: 



1. We always conveniently call` printf()` whenever we want to display our output to the console. `printf ` itself is a system call API. This function requires kernel service as it involves access to hardware, in particular the display unit. The function `printf`  is actually making several other function calls to _prepare the resources / requirements for this system call _and finally make the _actual_ system call` `that invokes the kernel’s help to display the output to the display. 



<p id="gdcalert7" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image7.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert8">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image7.png "image_tooltip")




2. We also always conveniently copy one file into another location (be it programmatically or through the GUI). In Win32 API, this is supported by the `CopyFile`[^7] function. The _actual _instruction sequence that is made by the function to complete the entire copy operation is actually pretty lengthy, involving multiple system calls (in this case alone system calls are made for writing to screen, opening files, obtaining file name, reading from file, termination, etc) - _image screenshot from SGG book._


<p id="gdcalert8" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image8.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert9">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image8.png "image_tooltip")



## System Call Implementation {#system-call-implementation}

An API helps users make appropriate system calls by providing convenient wrapper functions. Here’s the sequence of what happens in the nutshell after we invoke API functions involving system calls:



1. For most programming languages, the run-time support system (a set of functions built into libraries included with a compiler) provides a system call interface that serves as the link to system calls made available by the operating system. 
2. The system-call interface intercepts function calls in the API and invokes the necessary system calls within the operating system. 
3. Typically, a number is associated with each system call, and the system-call interface maintains a table indexed according to these numbers. For example, Linux system call tables and its associated numbers can be found here: [https://github.com/torvalds/linux/blob/master/arch/x86/entry/syscalls/syscall_64.tbl](https://github.com/torvalds/linux/blob/master/arch/x86/entry/syscalls/syscall_64.tbl)
4. The system call interface then invokes the intended system call in the operating-system kernel by interrupting itself and invoking the trap handler (runs in kernel mode from now onwards):
    1. Then for example, the trap handler saves the states of the process and examines the system call index left in a certain register.
    2. It then refers to the standard system call table and dispatches the system service request accordingly, i.e: branches onto the address in the Kernel space that implements the system call service routine of the system call with that index and executes it.
5. If the system call service routine returns to the trap handler, the program execution can be resumed. If the system call does not return, then the scheduler may be called to schedule another process, while this process is put to wait. 

In a nutshell, the function calls within the API reach the system call, the execution of the user application is temporarily suspended, since a system call is essentially a software interrupt. 

Note that the system call ids are HARDWARE DEPENDENT. It varies from systems to systems that utilise different hardwares. The relationship between application program, API, System call interface, and the kernel is shown below (_image screenshot from SGG book)_: 



<p id="gdcalert9" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image9.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert10">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image9.png "image_tooltip")


Example: making system call in C using API vs directly using assembly to print to screen 



<p id="gdcalert10" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image10.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert11">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image10.png "image_tooltip")


Typically we will do the above to print a simple hello, world!



<p id="gdcalert11" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image11.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert12">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image11.png "image_tooltip")


We can also do the same thing by using `write`. In fact, `write` is implemented within `printf` in C standard library as shown in the figure [earlier](#system-calls-through-api). For obvious reasons,  `printf` is more convenient because we don’t have to care about arguments like “1” and “14” and having to read the manual page for your hardware to find out what these mean inside `write`. 

But realise <code>write</code> here is just another C function call, not a system call. The figure in the previous page simply shows that a system call will be made within function <code>write.</code>

`write` is actually a convenient and more human-friendly wrapper around the system call `SYS_write`, and its implementation varies depending on the OS. The program above works on Linux and on macOS for this reason. 

Many wrappers in APIs are named after the system call itself, just like <code>write</code> here that’s meant to invoke<code> system call write.</code> This is the reason why many will not know whether they are making a direct system call or simply using the API to make a system call. 



<p id="gdcalert12" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image12.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert13">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image12.png "image_tooltip")


`write` is implemented by invoking another function called `syscall`[^8]. So we can do the same thing by calling `syscall` instead and passing the appropriate parameters. `SYS_write` here is actually the `id` of the system call that performs this write-to-console operation.

How is `syscall` implemented? `main` function puts its arguments in the right registers for the system call, and branch to `syscall` which is essentially composed of a bunch of assembly instruction:

You can completely make the system call for printing purposes directly using _C-inline assembly_. It is first done by putting the right arguments to the registers such as: `rax` -- the register to pass the system call id, `rdi` --  the register that should contain file descriptor (1 for `stdout` in your terminal), etc. Note that these registers are machine specific. These commands will only work on 64-bit Linux distro running on x86_64 architecture at the time of this writing. _It will compile on Mac systems but will not do anything since the registers are not correct for its hardware._



<p id="gdcalert13" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image13.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert14">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image13.png "image_tooltip")




The same print instruction on the Apple M1 chip (ARM-based) is as follows. You don’t have to learn them obviously, but we just want to show you that assembly language is _machine specific_:


```
.global _start             // Provide program starting address to linker
.align 2

// Setup the parameters to print hello world
// and then call Linux to do it.

_start: mov X0, #1     // 1 = StdOut
       adr X1, helloworld // string to print
       mov X2, #13     // length of our string
       mov X16, #4     // MacOS write system call
       svc 0     // Call linux to output the string

// Setup the parameters to exit the program
// and then call Linux to do it.

       mov     X0, #0      // Use 0 return code
       mov     X16, #1     // Service command code 1 terminates this program
       svc     0           // Call MacOS to terminate the program

helloworld:      .ascii  "hello, world!\n"
```



## 


## System Call Parameter Passing {#system-call-parameter-passing}

System call service routines are just like common functions, implemented in the kernel space. They require parameters to run. For example, if we request a `write`, one of the most obvious parameters required are the strings to write.  

There are three general ways to pass the parameters required for system calls to the OS Kernel:



1. Pass parameters in registers:
    * For the example of `write` system call, Kernel examines certain special registers for characters to print 
    * Pros: Simple and fast access
    * Cons: There might be more parameters than registers
2. Push parameters to the program stack:
    * Pushed to the stack by process running in user mode, then invoke` syscall(i)`
    * In kernel mode, pops the arguments from the program’s stack
3. Pass parameters that are stored in a _block_ or _table_ in RAM and pass the pointer through registers, to be read by the system call routine:
    * As illustrated below, `x` represents the address of the parameters for the system call. 
    * When system call `y` is made, the kernel examines certain registers, in this example is `R0` to obtain the address to the parameter
    * Given an address, Kernel can find the parameter for the system call in the RAM 




## Types of System Calls {#types-of-system-calls}

In general, each OS will provide a list of system calls that can be made. System calls can be grouped (but not limited to) roughly into six major categories: 



1. Process control: end, abort, load, execute, create and terminate processes, get and set process attributes, wait for time, wait for event, signal event, allocate, and free memory
2. File manipulation: create, delete, rename, open, close, read, write, and reposition files, get, and set file attributes
3. Device manipulation: request and release device, read from, write to, and reposition device, get and set device attributes, logically attach or detach devices
4. Information maintenance: get or set time and date, get or set system data, get or set process, file, or device attributes
5. Communications: create and delete pipes, send or receive packets through network, transfer status information, attach or detach remote devices, etc
6. Protection: set network encryption, protocol

If you are curious about Linux-specific system calls, you can find the list [here](http://asm.sourceforge.net/syscall.html). 

The screenshot below shows several examples of Windows and UNIX API functions that perform system calls  (_image screenshot from SGG book)_: 


### Process Control {#process-control}

In this section we choose to explain one particular type of system calls: process control with a little bit more depth. 



1. System calls to end or abort a program:

    A running process can either terminate normally (end) or abruptly (abort). 


    If a system call is made to terminate the currently running program abnormally, or if the program runs into a problem and causes an error trap, a dump of memory is sometimes taken and an error message generated. The dump is written to disk and may be examined by a debugger -- a type of system program. It is assumed that the user will issue an appropriate command to respond to any error. 

2. System calls to load and execute another program and to support process communication:
* It is possible for a process to call upon the execution of another process, such as creating background processes, etc. 
* Having created new jobs or processes, we may need to wait for them to finish their execution. 
* We may want to wait for a certain amount of time to pass `(wait time)`; more probably, we will want to wait for a specific event to occur` (wait event)`. 
* The jobs or processes should then signal when that event has occurred `(signal event)`. 
* Quite often, two or more processes share data and multiple processes need to communicate. All these features to `wait, signal event`, and communicate are done by making system calls. 

There are so many facets of and variations in process and job control that we need to clarify using examples:



1. Single-tasking system (e.g: MS-DOS,  _image screenshot from SGG book_): 
* The MS-DOS operating system is an example of a single-tasking system. 
* It has a command interpreter  (recall: part of [OS interface](#operating-system-user-interface)) that is invoked when the computer is started as shown in the figure above, labeled as (a). 
* Upon opening a new program, it loads the program into memory, writing over most of itself (note the shrinking portion of command interpreter codebase) to give the program as much memory as possible as shown in (b) above. 
* Next, it sets the instruction pointer to the first instruction of the program. 
    * The program then runs, and either an error causes a trap, or the program executes a system call to terminate. 
    * In either case, the error code is saved in the system memory for later use. 
* Following this action, the small portion of the command interpreter that was not overwritten resumes execution:
    * Its first task is to reload the rest of the command interpreter from disk. 
    * Then the command interpreter makes the previous error code available to the user or to the next program.
    * It stands by for more input command from the user.
2. Multi-tasking system (e.g:  FreeBSD,  _image screenshot from SGG book_) 

    The FreeBSD operating system is a multi-tasking OS that is able to create and manage multiple processes at a time:

* When a user logs on to the system, the shell of the user’s choice is run. 
* This shell is similar to the MS-DOS shell in that it accepts commands and executes programs that the user requests. 
* However, since FreeBSD is a multitasking system, the command interpreter may continue running while another program is executed: 
    * The possible state of a RAM with FreeBSD OS is as shown in the figure above  
    * To start a new process, the shell executes a` fork()` system call. 
    * Then, the selected program is loaded into memory via an `exec()`[^9] system call, and the program is executed normally until it executes `exit() `system call to end normally or `abort` system call. 


* Depending on the way the command was issued, the shell then either waits for the process to finish or runs the process “in the background.” 
    * In the latter case, the shell immediately requests_ another command. _


# 


# System Programs {#system-programs}

Apart from the Kernel and the user interface (GUI and/or CLI), a modern operating system also comes with system programs. Most users’ view of an operating system is defined by the system programs, not the actual system calls _because they are actually hidden from us (through API)_. 

System programs, also known as system utilities, provide a convenient environment for program development and execution:



1. They are basic tools used by many users for common low-level activities. 
2. It runs on user mode, just like any other user-level applications. When it requires kernel services, they make system calls just like any other user programs.
3. These tools are very generic, thus can be considered as part of the “system” instead of individual user apps that we typically install
4. _Note that sometimes system programs and system calls have the same name, but they are nowhere the same. For example: _
    1. `write` as a command that can be typed on the terminal (type `man write `to find out what these arguments are.)[^10]


    2. `write` system call as we have seen [above](#system-call-implementation)

Like system calls, they can also be divided into the following categories:



* File management:
    *  These programs create, delete, copy, rename, print, dump, list, and generally manipulate files and directories. 
    * For example: _all commands that you can enter in CLI that involves file management in UNIX systems is actually the name of system programs _
* Status information:
    * Some programs simply ask the system for the date, time, amount of available memory or disk space, number of users, or similar status information. 
    * Others are more complex, providing detailed performance, logging, and debugging information. 
    * Typically, these programs format and print the output to the terminal or other output devices or files or display it in a window of the GUI. Some systems also support a registry, which is used to store and retrieve configuration information.
* File modification:
    * Several text editors may be available to create and modify the content of files stored on disk or other storage devices. 
    * There may also be special commands to search the contents of files or perform transformations of the text.
* Programming-language support:
    * Compilers, assemblers, debuggers, and interpreters for common programming languages (such as C, C++, Java, and PERL) are often provided with the operating system or available as a separate download.
* Program loading and execution:
    *  Once a program is assembled or compiled, it must be loaded into memory to be executed. 
    * The system may provide absolute loaders, relocatable loaders, linkage editors, and overlay loaders. 
    * Runtime debugging systems for either higher-level languages or machine language are needed as well.
* Communications: 
    * These programs provide the mechanism for creating virtual connections among processes, users, and computer systems. 
    * They allow users to send messages to one another’s screens, to browse Web pages, to send e-mail messages, to log in remotely, or to transfer files from one machine to another.
    * _For example: ssh, pipe_

        _ _

* Background services:
    * All general-purpose systems have methods for launching certain system-program processes at boot time (upon startup): network-related system programs, some device drivers (although there are drivers that run in kernel mode, these are _not _system programs), etc  
    * Constantly running system-program processes are known as services, subsystems, or daemons. One example is the network daemon:
        * A system needed a service to listen for network connections in order to connect those requests to the correct processes. 
    * Other daemon examples include:  
        * The <code><em>init </em></code>process (systemd in Linux, launchd in macOS)
        * Process schedulers that start processes according to a specified schedule, 
        * System error monitoring services, 
        * Print servers, network services, etc
    * Typical systems have dozens of daemons. In addition, operating systems that run important activities in user mode rather than in kernel mode may use daemons to run these activities.


# 


# Application Programs {#application-programs}

Along with system programs, most operating systems are supplied with programs that are useful in solving common problems or performing common operations. Such application programs include Web browsers, word processors and text formatters, spreadsheets, database systems, compilers, plotting and statistical-analysis.packages, and games.

The table below summarises the differences between system programs and application programs (user programs)



<p id="gdcalert14" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image14.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert15">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image14.png "image_tooltip")


The view of the operating system seen by most users is defined by the application and system programs, rather than by the actual system calls. For example, when a user’s computer is running the Mac operating system, the user might see the GUI, featuring a mouse-and-windows interface. 


# 


# OS Design and Implementation {#os-design-and-implementation}

There’s no known ultimate solution when it comes to OS design. Internal structures of known operating systems can vary widely. 

When one design an OS, it might be helpful to consider a few known things:



1. Start by defining goals:
    1. User goals: OS should be convenient to use, easy to learn, reliable, safe, fast
    2. System goals: The system should be easy to design, implement, and maintain; and it should be flexible, reliable, error free, and efficient. 
2. Know the difference between policy and mechanism and separate them:
    3. Policy: determines what will be done
    4. Mechanism: determines how to do something
    5. The separation of policy and mechanism is important for flexibility:
        * Policies are likely to change across places or over time. 
        * In the worst case, each change in policy would require a change in the underlying mechanism. 
        * A general mechanism insensitive to changes in policy would be more desirable. A change in policy would then require _redefinition of only certain parameters of the system_. 
    6. For example: a mechanism for giving priority to certain types of programs over others. If the mechanism is properly separated from policy, it can be used either to
        * Support a policy decision that I/O-intensive programs should have priority over CPU-intensive ones 
        * Or support the opposite policy whenever appropriate.

Once an operating system is designed, it must be implemented. Because operating systems are _collections of many programs: kernel, system programs, interface, etc_, written by many people over a long period of time, it is difficult to make general statements about how they are implemented[^11].


## Example of OS structures {#example-of-os-structures}


### Monolithic Structure {#monolithic-structure}

A monolithic kernel is an operating system architecture where the entire operating system is working in kernel space. 

<span style="text-decoration:underline;">Without dual mode (simple):</span>

The figure below[^12] shows the structure of _MS-DOS,_ one of the simplest OS made in the early years:



* The interfaces and levels of functionality are not well separated (all programs can access the hardware) - i.e: at the time, MS-DOS was written for the Intel 8088 architecture, which has no mode bit and therefore no dual mode. 
* For instance, application programs are able to access the basic I/O routines to write directly to the display and disk drives.
* Such freedom leaves MS-DOS vulnerable to errant (or malicious) programs, causing entire system crashes when user programs fail.



<span style="text-decoration:underline;">With dual mode:</span>

The early UNIX OS was also simple in its form as shown below. In a way, it is layered to a minimal extent with very simple structuring. 



<p id="gdcalert15" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image15.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert16">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image15.png "image_tooltip")


The kernel provides file system management, CPU scheduling, memory management, and other operating-system functions through system calls. 

Taken in sum, that is an enormous amount of functionality to be combined into one level.

Pros:  _distinct performance advantage _because_ _there is very little overhead in the system call interface or in communication within the kernel. 

Cons:  _difficult to implement and maintain. _

Other examples: BSD, Solaris


### 


### Layered Approach {#layered-approach}

The operating system is broken into a many number of layers (levels). The bottom layer (layer 0) is the hardware; the highest (layer N) is the user interface. 

Figure below shows a layered approach (layer names for illustration purposes). The programs in layer N rely on services ONLY from the layer below it.



<p id="gdcalert16" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image16.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert17">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image16.png "image_tooltip")


Pros: 



* Simple to construct and debug
* Each layer is implemented only with operations provided by lower-level layers.
* A layer does not need to know how these operations are implemented; it needs to know only what these operations do. 
* This abstracts and hides the existence of certain data structures, operations, and hardware from higher-level layers.

 

Cons:



* Appropriately defining the various layers, and careful planning is necessary.
    * If we are met with bugs in our program, we debug our program and not our compiler
    * We mostly assume that the layers beneath us are already _made correct_
    * Sometimes, this assumption is not always true and difficult to maintain with the growing size of the OS. Some OS is shipped with bugs on its lower layers that are very difficult for users to debug. 
    * Patches and updates are periodically given to fix these bugs. 
* They tend to be less efficient than other types. 
    * For instance, when a process running in user mode executes an I/O operation, it executes a system call that is trapped to the I/O layer, which calls the memory-management layer, which in turn calls the CPU-scheduling layer, which is then passed to the hardware. 
    * At each layer, the parameters may be modified, data may need to be passed, and so on.
    * Each layer adds overhead to the system call. 
    * The net result is a system call that takes longer than does one on a non layered system.

Example: Windows NT (the later version is actually a _hybrid_ OS, combining between layered and monolithic aspects and benefits)[^13]


### 

<p id="gdcalert17" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image17.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert18">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image17.png "image_tooltip")



### 


### Microkernel structures {#microkernel-structures}

Microkernel is a very small kernel that provides minimal process and memory management, in addition to a communication facility.

This method structures the operating system by removing all nonessential components from the kernel and implementing them as system and user-level programs. The result is a _smaller kernel_ that does only tasks pertaining to:



1. IPC,
2. Memory Management,
3. Scheduling

Example: if the client program wishes to access a file, it must interact with the file server. The client program and service never interact directly. Rather, they communicate indirectly by exchanging messages with the microkernel as illustrated below:



<p id="gdcalert18" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image18.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert19">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image18.png "image_tooltip")


Pros: extending the operating system easier. All new services are added to user space and consequently do not require modification of the kernel. 

Cons: suffer in performance  increased system-function overhead due to frequent requirement in performing context switch. 

Example: Mach, Windows NT (first release was a microkernel).


### 


### Hybrid Approach {#hybrid-approach}

Hybrid kernels attempt to combine between microkernel and monolithic kernel aspects and benefits. 

Example:

macOS is partly based on microkernel + monolithic approach  _(image taken from SGG)_:



1. Mach provides: IPC, scheduling, memory management
2. BSD provides: CLI, file system management, networking support, POSIX APIs implementations



<p id="gdcalert19" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image19.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert20">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image19.png "image_tooltip")



### 


### Java Operating System (JX) {#java-operating-system-jx}

The JX OS is written almost entirely in Java. Such a system, known as a language-based extensible system, and runs in a single address space (no virtualisation, no MMU), as such it will face difficulties in maintaining memory protection that is usually supported by hardwares in typical OS.



<p id="gdcalert20" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image20.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert21">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image20.png "image_tooltip")


_Language-based systems instead rely on type-safety_[^14]_ features of the language. As a result, language-based systems are desirable on small hardware devices, which may lack hardware features that provide memory protection._

Since Java is a type-safe language, JX is able to provide isolation between running Java applications without hardware memory protection. This is called _language based protection, _where system calls and IPC in JX does not  require an address-space switch. In short, JX runs in a single address space.  

 \


The architecture of the JX system is illustrated below (simplified representation[^15]):



* JX organizes its system according to domains. 



<p id="gdcalert21" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image21.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert22">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image21.png "image_tooltip")




* Each domain represents an independent JVM (Java Virtual Machine):
    * JVM is an abstract virtual machine that can run on any OS
    * There’s one instance of JVM per Java application
    * JVM provides portable execution environment for Java-based apps
    * It maintains a heap used for allocating memory during object creation and threads within itself, as well as for garbage collection. 



<p id="gdcalert22" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image22.jpg). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert23">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image22.jpg "image_tooltip")




* Domain zero is a microkernel responsible for_ low-level details_, such as system initialization and saving and restoring the state of the CPU. 
    * Domain zero is written in C and assembly language; all other domains are written entirely in Java. 
    * Communication between domains occurs through a specific mechanism called _portals_
* Protection within and between domains relies on the type safety of the Java language. However, since domain zero is not written in Java, it must be considered trusted. 


## Summary {#summary}

The figure below shows the summary of various OS structures. 



<p id="gdcalert23" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image23.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert24">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image23.png "image_tooltip")


For layered architecture, note that the only difference is that programs at level N relies ONLY on services provided by programs at one level below it. 


[^2]:
     Image taken from https://notes.shichao.io/tlpi/ch6/

[^3]:
     A library is a chunk of code that _implements an API_. An API (application programming interface) is a term that refers to the functions/methods in a library that you can call to perform the task on your behalf (without you actually having to implement the code). As its name said, an API for a particular library, is the _interface _to the library. The same API can be implemented by different libraries (implementation). The underlying libraries can be updated, etc without changing the API and hence not _breaking _other code that utilizes the API.

[^4]:

     Actual system calls can often be more detailed and difficult to work with than the API available to an application programmer.

[^5]:

     An application programmer designing a program using an API can expect her program to compile and run on any system that supports the same API (although in reality, architectural differences often make this more difficult than it may appear)

[^6]:
     Linux is a Unix clone written from scratch by Linus Torvalds with assistance from a loosely-knit team of hackers across the Net. It aims towards POSIX compliance.

[^7]:

    [https://docs.microsoft.com/en-gb/windows/win32/api/winbase/nf-winbase-copyfile?redirectedfrom=MSDN](https://docs.microsoft.com/en-gb/windows/win32/api/winbase/nf-winbase-copyfile?redirectedfrom=MSDN)

[^8]:
     From the documentation: _ syscall() is a small library function that invokes the system call whose assembly language interface has the specified number with the specified arguments. syscall() in Linux saves CPU registers before making the system call, restores the registers upon return from the system call, and stores any error code returned by the system call in [errno(3)](https://manpages.debian.org/unstable/manpages-dev/errno.3.en.html) if an error occurs._

[^9]:

     We will learn about these system calls in the latter weeks and in lab

[^10]:

      `tty` itself is a command in Unix and Unix-like operating systems to print the file name of the terminal connected to standard input. If you open multiple terminal windows in your UNIX-based system and type `tty` on each of them, you will be returned with different ids. You may use write to communicate across terminal windows

[^11]:
     Early operating systems were written in assembly language. Now, although some operating systems are still written in assembly language, most are written in a higher-level language such as C or an even higher-level language such as C++. Actually, an operating system can be written in more than one language: (1) The lowest levels of the kernel might be assembly language, and then (2) higher-level routines might be in C, and finally (3) system programs might be in C or C++, in interpreted scripting languages like PERL or Python, or in shell scripts. In fact, a given Linux distribution probably includes programs written in all of those languages.

[^12]:
     _image screenshot from SGG book_

[^13]:
     Figure taken from Wikipedia

[^14]:
    The Java language is designed to enforce type safety. This means that programs are prevented from accessing memory in inappropriate ways. Every section of memory is part of some Java object that belongs to some class. For example, a calendar-management applet might use classes like Date, Appointment, Alarm, and GroupCalendar. Each class defines both a set of objects and operations to be performed on the objects of that class (explanation taken from Securing Java (ISBN: 047131952X), section 10)

[^15]:
     Image taken from https://webmobtuts.com/news-events/what-is-the-jvm-introducing-the-java-virtual-machine/
