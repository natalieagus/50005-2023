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

1. <span style="color:#f77729;"><b>Hardware</b></span>, e.g.: PC cannot perform `JMP` to code with raw RAM addresses starting with MSB of ‘1’
2. <span style="color:#f77729;"><b>Virtualisation</b></span>, user programs operate on virtual memory

System calls are mostly accessed by programs through <span style="color:#f7007f;"><b>APIs</b></span> (application program interface), although we can certainly make system calls directly in assembly. There are plenty of examples in the next few sections.

A system call does <span style="color:#f7007f;"><b>NOT</b></span> generally require a <span style="color:#f7007f;"><b>full</b></span> context switch; instead, it is processed in the context of whichever process invoked it.

## Accessing System Calls {#extras-accessing-system-calls}

Take this section with a grain of salt. It is not part of the syllabus but it is handy for you to understand how a regular user process can access kernel code via system calls.
{:.warning}

All processes running in a computer must be able to make system calls. As a result, at the minimum the <span style="color:#f7007f;"><b>entry</b></span> points into the kernel have to be <span style="color:#f77729;"><b>mapped</b></span> into the current address space at all times.

To provide you with a context, let’s see a typical memory layout[^2] of a UNIX process (actual implementation may vary, e.g. Kernel is at low address space instead):
<img src="/50005-2023/assets/images/week2/1.png"  class="center_full"/>

The virtual address space of a process is typically divided into two parts: <span style="color:#f77729;"><b>kernel</b></span> part in the higher address and <span style="color:#f77729;"><b>user</b></span> part in the lower address (or vice versa, depending on the Kernel implementation).

The kernel mapping part exists primarily for the<span style="color:#f77729;"><b> kernel-related purposes</b></span>, not user processes. Processes running in user mode don’t have access to the kernel’s address space (with different MSB), at all. In user mode, there is a <span style="color:#f77729;"><b>single</b></span> mapping for the kernel, shared across all processes, e.g: fixed address between `0xffffffff` to `0xC0000000` as illustrated above.

- When a kernel-side page mapping changes, that change is reflected everywhere.

### Kernel Space

The kernel is divided into two spaces: <span style="color:#f77729;"><b>logical</b></span> and <span style="color:#f77729;"><b>virtual</b></span>, often called `lowmem` and `vmalloc` respectively.

In `lowmem`, it often uses a <span style="color:#f77729;"><b>one-to-one</b></span> mapping between virtual and physical addresses (its called logical mapping). That means virtual address `X` is mapped to physical address `X+C` (where `C` is some constant if any). This mapping is built during <span style="color:#f77729;"><b>boot</b></span>, and is <span style="color:#f77729;"><b>never</b></span> changed.

The kernel virtual address area (`vmalloc`) is used for <span style="color:#f77729;"><b>non-contiguous</b></span> physical memory location, so that it is <span style="color:#f77729;"><b>easier</b></span> to allocate them.

- This allocation of process memory is <span style="color:#f77729;"><b>dynamic</b></span> and on demand.
- On each allocation, a series of locations of physical pages are found for the corresponding kernel virtual address range, and the pagetable is <span style="color:#f77729;"><b>modified</b></span> to create the mapping.
- If this is done, it might be <span style="color:#f77729;"><b>unsuitable</b></span> for DMA (Direct Memory Access).

# System Calls via API {#system-calls-through-api}

One of the most common ways to make system calls is through an <span style="color:#f7007f;"><b>API</b></span> (Application Programming Interface). We can write a program in a particular language and conveniently perform several system calls through the provided APIs supported by the chosen language as needed.

API is an <span style="color:#f77729;"><b>interface</b></span> that provides a way to interact with the underlying library[^3] that makes the system calls, often <span style="color:#f77729;"><b>named the same</b></span> as the system calls they invoke.
{:.warning}

An API specifies:

- a set of functions that are available to an application programmer
- the parameters that are passed to each function and
- the return values the programmer can expect

3 most common APIs available:

1. Win32 API for Windows systems written in C++
2. POSIX API for POSIX-based system (all versions of UNIX); mostly written in C. You can find the functions supported by the API [here](https://pubs.opengroup.org/onlinepubs/9699919799/)
3. Java API for programs running on Java Virtual Machine (JVM)

Behind the scenes, the **functions** that make up an API invoke the **actual** system calls on behalf of the application programmer.

<img src="/50005-2023/assets/images/week2/2.png"  class="center_seventy"/>

Benefits of using an API to make system calls:

- It adds another layer of **abstraction** hence simplifies the process of application development[^4]
- Supports program **portability**[^5]

### Example: printf()

We always conveniently call `printf()` whenever we want to display our output to the console in C. `printf` itself is a POSIX system call <span style="color:#f77729;"><b>API</b></span>.

- This function requires kernel service as it involves access to hardware: output display.
- The function `printf` is actually making several other function calls to prepare the resources or requirements for this system call **and** finally make the actual system call that invokes the kernel’s help to display the output to the display.

<img src="/50005-2023/assets/images/week2/3.png"  class="center_fifty"/>

The full implementation of `printf` in Mach OS can be found [here](https://opensource.apple.com/source/xnu/xnu-201/osfmk/kern/printf.c.auto.html). It calls other functions like `putc` and eventually `write` function that makes the system call to `stdout` file descriptor.

### Example: CopyFile()

We also always conveniently copy one file into another location (be it programmatically or through the GUI). In Windows OS, this is supported by the `CopyFile`[^7] function (Win32 API):

```
CopyFile(szFilePath.c_str(), szCopyPath.c_str(), FALSE );
```

The complete documentation can be found [here](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-copyfile). The instruction sequence that is made by this `CopyFile` function to complete the entire copy operation is actually pretty lengthy, involving **multiple** system calls. In `CopyFile` case alone, multiple system calls are made for writing to file, opening files, obtaining file name, reading from file, termination, etc (image below taken from SGG book)

<img src="/50005-2023/assets/images/week2/4.png"  class="center_seventy"/>

The actual implementation details (source code) of OS functions like `CopyFile` are intentionally not documented and can be changed at any time. The API however is well documented and conformed to so others who rely on it will not have their programs broken due to internal OS updates.
{:.warning}

## System Call Implementation {#system-call-implementation}

An API helps users make appropriate system calls by providing convenient wrapper functions. More often than not, we don't need to know its detailed implementation as the API already provides convenient <span style="color:#f77729;"><b>abstraction</b></span>.

For most programming languages, the <span style="color:#f77729;"><b>run-time support system</b></span> (a set of functions built into libraries included with a compiler) provides a system call <span style="color:#f77729;"><b>interface</b></span> that serves as the link to system calls made available by the operating system. The system-call <span style="color:#f77729;"><b>interface</b></span> intercepts function calls in the API and invokes the necessary system calls within the operating system.

### System Call Number

Typically, a <span style="color:#f77729;"><b>number</b></span> is associated with each system call, and the system-call interface maintains a table indexed according to these numbers. For example, Linux system call tables and its associated numbers can be found [here](https://github.com/torvalds/linux/blob/master/arch/x86/entry/syscalls/syscall_64.tbl) and FreeBSD Kernel System call table can be found [here](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master).

To further understand the points above, let's see a sample assembly file to print `Hello World` to console.

#### Hello World in Assembly (Linux, x86-64)

```armasm
; hello_world.s
global _start

section .text

_start:
  mov rax, 1        ; system call number for write
  mov rdi, 1        ;    making file handle stdout
  mov rsi, msg      ;   passing adress of string to output
  mov rdx, msglen   ;   number of bytes
  syscall           ; invoking os to write

  mov rax, 60       ; sys call number for exit
  mov rdi, 0        ;   exit code 0 EXIT_SUCCESS
  syscall           ; invoke os to exit

section .rodata
  msg: db "Hello, world!", 10
  msglen: equ $ - msg
```

Compilation and execution:

```bash
nasm -f elf64 -o hello_world.o hello_world.s
ld -o hello_world hello_world.
./hello_world
```

#### Hello World in Assembly (macOS X, x86-64)

```armasm
; hello_world.s
global _main
section .text
_main:
  mov rax, 0x2000004 ; system call number for write
  mov rdi, 1 ; file descriptor 1 is stdout
  mov rsi, msg ; get string address
  mov rdx, msg.len ; number of bytes
  syscall ; exec syscall write
  mov rax, 0x2000001 ; syscall number for exit
  mov rdi, 0 ; exit code 0
  syscall ; exit program
section .data
msg:    db      "Hello, world!", 10
.len:   equ     $ - msg
```

Compilation and execution:

```bash
nasm -f macho64 hello_world.s
ld -lSystem -o hello_world hello_world.o
./hello_world
```

#### Hello World in Assembly (macOS X, M1)

```armasm
// hello_world.s
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

Compilation and execution:

```bash
as -g -o hello_world.o hello_world.s
ld -macosx_version_min 12.0.0 -o hello_world hello_world.o -lSystem -syslibroot `xcrun -sdk macosx --show-sdk-path` -e _start -arch arm64
./hello_world
```

From the three examples above, it is obvious that system call numbers are <span style="color:#f7007f;"><b>hardware dependent</b></span>. Note that the <span style="color:#f77729;"><b>registers used</b></span> are also machine specific. In x86-64 architecture, `rax` stands for the register to pass the system call id, `rdi` stands the register that should contain file descriptor (1 for `stdout` in your terminal), etc. In M1 architecture, the equivalent to `rax` is `X16`, and `rdi` is `X0`.

#### Hello World in C

In contrast, here's an implementation in C, short and sweet.

```c
// hello_world.c
#include <stdio.h>
int main() {
   // printf() displays the string inside quotation
   printf("Hello, World!");
   return 0;
}
```

Compilation and execution:

```bash
gcc -o hello_world hello_world.c
./hello_world
```

Here's the implementation in Python. Even shorter and sweeter:

```python
# hello_world.py
print('Hello, world!')
```

Execution: `python hello_world.py`

As you can see, the C program can be compiled for either OS, and so is the python program. The C system call <span style="color:#f77729;"><b>interface</b></span> then invokes the intended system call in the operating-system kernel by <span style="color:#f77729;"><b>trapping</b></span> itself and invoking the trap handler (runs in kernel mode from now onwards):

1. The trap handler first <span style="color:#f77729;"><b>saves</b></span> the states of the process[^8] and examines the system call index left in a certain register.
2. It then refers to the standard system call table and <span style="color:#f77729;"><b>dispatches</b></span> the system service request accordingly, i.e: <span style="color:#f77729;"><b>branches</b></span> onto the address in the Kernel space that implements the system call service routine of the system call with that index and executes it.

When the system call service routine <span style="color:#f77729;"><b>returns</b></span> to the trap handler, the program execution can be <span style="color:#f77729;"><b>resumed</b></span>. If the system call does not return yet (e.g: block system call like `input()`), then the scheduler may be called to schedule another process, while this process is put to wait until the requested service is available.

The relationship between application program, API, System call interface, and the kernel is shown below (_image screenshot from SGG book)_:

<img src="/50005-2023/assets/images/week2/5.png"  class="center_seventy"/>

Notice that `printf` is just a C function that will eventually calls the `write` C function, and will eventually invoke the `write` systemcall. We can also do the same thing by using `write` C function:

```c
// hello_world.c
#include <unistd.h>
int main()
{
    // write(fd, char*, #bytes)
    write(1, "hello, world!\n", 14);
    return 0;
}
```

For obvious reasons, `printf` is more convenient because we don’t have to care about arguments like “1” (file descriptor for stout) and “14” (bytes in the printed string) and having to read the manual page for your hardware to find out what these mean inside `write`.

`write` is actually a convenient and more human-friendly <span style="color:#f77729;"><b>wrapper</b></span> around the another C function: `syscall`. It utilises the symbol `SYS_write` to indicate the system call number which value varies depending on the OS.

```c
#include <unistd.h>
#include <sys/syscall.h>

int main()
{
    // write(fd, char*, #bytes)
    syscall(SYS_write, 1, "hello, world!\n", 14);
    return 0;
}
```

Many wrappers in APIs are named after the system call itself, just like `write` or `syscall` that’s meant to invoke the actual WRITE system call and invoke the `syscall` routine.
In summary, making system calls <span style="color:#f77729;"><b>directly</b></span> in the application code is possible, but more complicated and may require embedded assembly code to be used (in C and C++) as well as knowledge of the low-level binary interface for the system call operation, which may be subject to <span style="color:#f77729;"><b>change</b></span> over time and thus not be part of the application binary interface; <span style="color:#f7007f;"><b>the API is meant to abstract this away</b></span>.
{:.warning}

You can find out more about Linux[^6] system calls <span style="color:#f77729;"><b>API</b></span> (implemented in C) [here](http://man7.org/linux/man-pages/man2/syscalls.2.html)

### Parameter Passing {#system-call-parameter-passing}

System call service <span style="color:#f77729;"><b>routines</b></span> are just like common functions, implemented in the kernel space. <span style="color:#f77729;"><b>We will do a little exercise with BSim soon to understand better.</b></span> They require <span style="color:#f77729;"><b>parameters</b></span> to run. For example, if we request a `write`, one of the most obvious parameters required are the bytes to write.

There are three general ways to pass the parameters required for system calls to the OS Kernel.

#### Registers

Pass parameters in <span style="color:#f77729;"><b>registers</b></span>:

- For the example of `write` system call, Kernel examines certain special registers for bytes to print
- Pros: Simple and <span style="color:#f77729;"><b>fast</b></span> access
- Cons: There might be <span style="color:#f77729;"><b>more</b></span> parameters than registers

#### Stack

Push parameters to the program <span style="color:#f77729;"><b>stack</b></span>:

- Pushed to the stack by process running in user mode, then invoke `syscall`
- In kernel mode, <span style="color:#f77729;"><b>pops</b></span> the arguments from the calling program’s stack

#### Block or Table

Pass parameters that are stored in a persistent contiguous location (<span style="color:#f77729;"><b>table</b></span> or <span style="color:#f77729;"><b>block</b></span>) in the RAM (this is a <span style="color:#f77729;"><b>different</b></span> location from stack!) and pass the <span style="color:#f77729;"><b>pointer</b></span> (address) through registers, to be read by the system call routine:

- As illustrated below, `x` represents the <span style="color:#f77729;"><b>address</b></span> of the parameters for the system call.
- When system call `id` (e.g: `write`) is made, the kernel examines certain registers, in this example is `rsi` to obtain the address to the parameter (the bytes to write to `stdout`)
- Given the <span style="color:#f77729;"><b>pointer</b></span>, Kernel can find the parameter for the system call in the RAM, as illustrated below:

<img src="/50005-2023/assets/images/week2/6.png"  class="center_seventy"/>

# Types of System Calls {#types-of-system-calls}

In general, each OS will provide a <span style="color:#f77729;"><b>list</b></span> of system calls that it supports. System calls can be grouped (but not limited to) roughly into six major categories:

1. <span style="color:#f7007f;"><b>Process control</b></span>: end, abort, load, execute, create and terminate processes, get and set process attributes, wait for time, wait for event, signal event, allocate, and free memory
2. <span style="color:#f7007f;"><b>File manipulation</b></span>: create, delete, rename, open, close, read, write, and reposition files, get, and set file attributes
3. <span style="color:#f7007f;"><b>Device manipulation</b></span>: request and release device, read from, write to, and reposition device, get and set device attributes, logically attach or detach devices
4. <span style="color:#f7007f;"><b>Information maintenance</b></span>: get or set time and date, get or set system data, get or set process, file, or device attributes
5. <span style="color:#f7007f;"><b>Communication</b></span>: create and delete pipes, send or receive packets through network, transfer status information, attach or detach remote devices, etc
6. <span style="color:#f7007f;"><b>Protection</b></span>: set network encryption, protocol

If you are curious about Linux-specific system call types, you can find the list [here](http://asm.sourceforge.net/syscall.html).

# Blocking vs Non-Blocking System Call

A <span style="color:#f7007f;"><b>blocking</b></span> system call is one that must <span style="color:#f7007f;"><b>wait</b></span> until the action can be completed.

For instance, `read()` is <span style="color:#f77729;"><b>blocking</b></span>:

- If no input is ready, the calling process will be <span style="color:#f77729;"><b>suspended</b></span>
  - `yield()` the remaining quanta, and schedule other processes first
- It will only resume execution after some input is ready. Depending on the scheduler implementation it may either:
  - Be scheduled again and <span style="color:#f77729;"><b>retry</b></span> (e.g: round robin)
    - The process re-executes `read()` and may `yield()` again if there's no input.
    - Repeat until successful.
  - <span style="color:#f77729;"><b>Not</b></span> scheduled, use some `wait` flag/status to tell the scheduler to not schedule this again unless some input is received
    - `wait` flag/status cleared by interrupt handler (more info in the next topic)

On the other hand, a <span style="color:#f7007f;"><b>non blocking</b></span> system call can return almost immediately without waiting for the I/O to complete.

For instance, [`select()`](https://linux.die.net/man/2/select) is non-blocking.

- The `select()` system call can be used to <span style="color:#f77729;"><b>check</b></span> if there is new data or not, e.g: at `stdin` file descriptor.
- Then a blocking system call like `read()` may be used afterwards knowing that they will complete immediately.

## Process Control {#process-control}

In this section we choose to explain one particular type of system calls: <span style="color:#f7007f;"><b>process control</b></span> with a little bit more depth.

### Process Abort

A running process can either <span style="color:#f77729;"><b>terminate</b></span> <span style="color:#f7007f;"><b>normally</b></span> (end) or <span style="color:#f7007f;"><b>abruptly</b></span> (abort). In either case, system call to <span style="color:#f77729;"><b>abort</b></span> a process is made.

If a system call is made to terminate the currently running program <span style="color:#f77729;"><b>abnormally</b></span>, or if the program runs into a problem and causes an error <span style="color:#f77729;"><b>trap</b></span>, a dump of memory (called [`core dump`](https://en.wikipedia.org/wiki/Core_dump)) is sometimes taken and an error message generated.

It consists of the recorded state of the program memory at that specific time when the program <span style="color:#f77729;"><b>crashed</b></span>. The dump is written to disk and may be examined by a debugger; a type of system program. It is assumed that the user will issue an appropriate command to respond to any error.

### Process Load and Execute

Loading and executing a new process in the system require system calls. It is possible for a process to call upon the execution of another process, such as creating background processes, etc.

- For instance the <span style="color:#f77729;"><b>shell</b></span> creates a new process whenever it receives a new command, and requests to execute that command in the <span style="color:#f77729;"><b>new process</b></span> (next chapter)

### Process Communication

Having created new jobs or processes, we may need to <span style="color:#f77729;"><b>wait</b></span> for them to <span style="color:#f77729;"><b>finish</b></span> their execution, e.g: the shell only gives the next prompt after the previous command has completed its execution.

- We may want to wait for a certain amount of time to pass `(wait time)`; more probably, we will want to wait for a specific event to occur` (wait event)`.
- The jobs or processes should then signal when that event has occurred `(signal event)`.
- Also, sometimes two or more processes <span style="color:#f77729;"><b>share</b></span> data and multiple processes need to <span style="color:#f77729;"><b>communicate</b></span> (e.g: a web server communicating with the database server).
- All these features to `wait, signal event`, and other means of process <span style="color:#f77729;"><b>communication</b></span> are done by making system calls since each process is run in <span style="color:#f77729;"><b>isolation</b></span> by default, operating on <span style="color:#f77729;"><b>virtual addresses</b></span>.

### Examples

There are so many facets of and variations in process and job control that we need to clarify using examples: MS-DOS and FreeBSD.

<img src="/50005-2023/assets/images/week2/7.png"  class="center_seventy"/>

#### Single-tasking System

An example of a single-tasking system is MS-DOS, shown in the figure on the left.

It has a simple command <span style="color:#f77729;"><b>interpreter</b></span> (that is invoked when the computer is started as shown in the figure above, labeled as `(a)`). Upon opening a new program, it <span style="color:#f77729;"><b>loads</b></span> the program into memory, <span style="color:#f77729;"><b>writing over most of itself</b></span> (note the shrinking portion of command interpreter codebase) to give the program<span style="color:#f77729;"><b> as much memory as possible </b></span>as shown in `(b)` above.

Next, it sets the <span style="color:#f77729;"><b>instruction pointer</b></span> to the first instruction of the program.

- The program then runs, and either an <span style="color:#f77729;"><b>error</b></span> causes a <span style="color:#f77729;"><b>trap</b></span>, or the program executes a system call to <span style="color:#f77729;"><b>terminate</b></span>.
- In either case, the error code is saved in the system memory for later use.

Following this action, the <span style="color:#f77729;"><b>small</b></span> portion of the command interpreter that was not overwritten resumes execution:

- Its first task is to reload the rest of the command interpreter from disk.
- Then the command interpreter makes the previous error code available to the user or to the next program.
- It stands by for more input command from the user.

#### Multi-tasking system

An example of a multi-tasking system is FreeBSD. The FreeBSD operating system is a <span style="color:#f77729;"><b>multi-tasking</b></span> OS that is able to create and manage multiple processes at a time.

When a user logs on to the system, the <span style="color:#f77729;"><b>shell</b></span> (command interpreter) of the user’s choice is run. This shell is similar to the MS-DOS shell in that it accepts commands and executes programs that the user requests.

However, since FreeBSD is a multitasking system, the command interpreter may <span style="color:#f77729;"><b>continue running</b></span> while another program is executed:

- The possible state of a RAM with FreeBSD OS is as shown in the figure above
- To <span style="color:#f77729;"><b>start</b></span> a new process, the shell executes a `fork()` system call.
- Then, the selected program is loaded into memory via an `exec()`[^9] system call, and the program is executed normally until it executes `exit() `system call to end normally or `abort` system call.

Depending on the way the command was issued, the shell then either <span style="color:#f77729;"><b>waits</b></span> for the process to finish or runs the process “in the background.”

> In the latter case, the shell immediately requests another command.

The kernel is responsible to ensure that <span style="color:#f77729;"><b>context switching</b></span> is properly done (and <span style="color:#f77729;"><b>timesharing</b></span> as well if enabled).

To run a command in the background, add the ampersand symbol (`&`) at the end of the command:

```bash
command &
```

[^2]: Image taken from [here](https://notes.shichao.io/tlpi/ch6/).
[^3]: A library is a chunk of code that implements an API. An API (application programming interface) is a term that refers to the functions/methods in a library that you can call to perform the task on your behalf (without you actually having to implement the code). As its name said, an API for a particular library, is the interface to the library. The same API can be implemented by different libraries (implementation). The underlying libraries can be updated, etc without changing the API and hence not breaking other code that utilizes the API.
[^4]: Actual system calls can often be more detailed and difficult to work with than the API available to an application programmer.
[^5]: An application programmer designing a program using an API can expect her program to compile and run on any system that supports the same API (although in reality, architectural differences often make this more difficult than it may appear)
[^6]: Linux is a Unix clone written from scratch by Linus Torvalds with assistance from a loosely-knit team of hackers across the Net. It aims towards POSIX compliance.
[^7]: [Refer to this document](https://docs.microsoft.com/en-gb/windows/win32/api/winbase/nf-winbase-copyfile?redirectedfrom=MSDN)
[^8]: From the documentation: `syscall()` is a small library function that invokes the system call whose assembly language interface has the specified number with the specified arguments. `syscall()` in Linux saves CPU registers before making the system call, restores the registers upon return from the system call, and stores any error code returned by the system call in [errno(3)](https://manpages.debian.org/unstable/manpages-dev/errno.3.en.html) if an error occurs.
[^9]: We will learn about these system calls in the latter weeks and in lab
