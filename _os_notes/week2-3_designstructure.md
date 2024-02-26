---
title: OS Design and Structure
permalink: /os_notes/week2_designstructure
key: os-notes-week2_designstructure
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

Apart from the Kernel and the user interface (GUI and/or CLI), a modern operating system also comes with <span style="color:#f77729;"><b>system programs</b></span> and <span style="color:#f77729;"><b>application programs</b></span>.

Most users’ view of an operating system is defined by the system programs, not the actual system calls because they are actually hidden from us (through API).
{:.warning}

For example, when a user’s computer is running the macOS, the user might see the GUI, featuring a mouse-and-windows interface.

# System Programs

System programs, also known as system <span style="color:#f77729;"><b>utilities</b></span>, provide a convenient environment for program development and execution:

1. They are <span style="color:#f77729;"><b>basic</b></span> tools used by many users for common low-level activities.
2. These tools are very <span style="color:#f77729;"><b>generic</b></span>, thus can be considered as part of the “system” instead of individual user apps that we typically install
3. Note that sometimes system programs and system calls have the same name, but they are nowhere the same. For example:
   1. `write` as a command that can be typed on the terminal (type `man write `to find out what these arguments are.)[^10]
   2. `write` is also a system call API and an actual system call name

System programs runs on <span style="color:#f7007f;"><b>user mode</b></span>, just like any other user-level applications. When it requires kernel services, they make system calls just like any other user programs.
{:.error}

## Categories

Like system calls, system programs can also be divided into the categories below.

### Package Managers: file management and modification

These programs create, delete, copy, rename, print, dump, list, and generally manipulate files and directories. For example: all commands that you can enter in CLI that involves file management in UNIX systems is actually the <span style="color:#f77729;"><b>name</b></span> of system programs that can be found in the `$PATH`. These include `ls, rm, mkdir, cp, touch` among many others.

Modern OS usually comes with default <span style="color:#f77729;"><b>package managers</b></span> that simplifies installation of software applications (and also managing versions, updates, running background services, etc). For instance, `brew` for macOS and `apt` for Debian-based Linux distributions.

Several text editors provided (`nano`, `vi`) may be available to create and modify the content of files stored on disk or other storage devices. There may also be special commands to search the contents of files or perform transformations of the text like `grep, awk, tr`.

### Status information

Some programs simply ask the system for various status <span style="color:#f77729;"><b>information</b></span>: date, time, amount of available memory or disk space, number of users, etc. Others are more complex, providing detailed performance, logging, and debugging information. Example include `top, ls, df` among many others.

Typically, these programs format and print the output to the terminal or other output devices or files or display it in a window of the GUI. Some systems also support a <span style="color:#f77729;"><b>registry</b></span>, which is used to store and retrieve system <span style="color:#f7007f;"><b>configuration</b></span> information.

### Programming-language support

<span style="color:#f77729;"><b>Compilers</b></span>, <span style="color:#f77729;"><b>assemblers</b></span>, <span style="color:#f77729;"><b>debuggers</b></span>, and <span style="color:#f77729;"><b>interpreters</b></span> for common programming languages (such as C, C++, Java, and Python) are often provided with the operating system or available as a separate download. Package managers such as `npm, pip` may be installed in modern OS to make it easier for users to develop.

Take these example of system programs with a grain of salt. The definitions of what program belongs to system program category is subject to debate. Some Operating Systems don't come with compilers and debuggers so they may not be categorised as "system programs" for some.
{:.warning}

### Program loading and execution

Once a program is assembled or compiled, it must be <span style="color:#f77729;"><b>loaded</b></span> into memory to be executed. The system may provide absolute <span style="color:#f77729;"><b>loaders</b></span>, relocatable loaders, <span style="color:#f77729;"><b>linkage</b></span> editors, and overlay loaders. Runtime debugging systems for either higher-level languages or machine language are needed as well.

### Communications

These programs provide the mechanism for creating <span style="color:#f77729;"><b>virtual</b></span> connections among processes, users, and computer systems. They allow users to send messages to one another’s screens, to browse Web pages, to send e-mail messages, to log in remotely, or to transfer files from one machine to another. Example include `ssh, pipe (|)`.

### Background services

All general-purpose systems have methods for <span style="color:#f77729;"><b>launching</b></span> certain system-program processes at boot time (upon startup): network-related system programs, some device drivers (although there are drivers that run in kernel mode, these are not system programs), etc

- Constantly running system-program processes are known as <span style="color:#f77729;"><b>services</b></span>, <span style="color:#f77729;"><b>subsystems</b></span>, or <span style="color:#f77729;"><b>daemons</b></span>.
- One example is the network daemon:
  - A system needed a service to <span style="color:#f77729;"><b>listen</b></span> for network connections in order to connect those requests to the correct processes.
- Other daemon examples include:
  - The `init` process (specifically called`systemd` in Linux, `launchd` in macOS)
  - Process <span style="color:#f77729;"><b>schedulers</b></span> that start processes according to a specified schedule,
  - System <span style="color:#f77729;"><b>error monitoring</b></span> services,
  - Print <span style="color:#f77729;"><b>servers</b></span>

Typical systems have <span style="color:#f77729;"><b>dozens</b></span> of daemons. In addition, operating systems that run important activities in user mode rather than in kernel mode may use daemons to run these activities.

# Application (User) Programs {#application-programs}

Along with system programs, most operating systems are supplied with programs that are useful for specific users. Such application programs are Web browsers, word processors and text formatters, spreadsheets, database systems, compilers, plotting and statistical-analysis.packages, and games.

# System vs Application Programs

It is often hard to distinguish between system and application (user) programs. Some programs like compiler, assembler, debugger, device drivers, and antivirus can be clearly defined as <span style="color:#f77729;"><b>system programs</b></span>. Programs like media player, photo editing software applications, and video games are clear examples of application programs.

The table below summarises the differences between system programs and application programs (user programs).

| System Programs                                                            | User Programs                               |
| -------------------------------------------------------------------------- | ------------------------------------------- |
| Used for operating computer hardware and very common system-usage purposes | Used to perform specific user-related tasks |
| Typically comes with the OS                                                | Installed according to user requirements    |
| Commonly runs in the background, require minimal to no user interactions   | Requires interactions with users            |

# OS Design and Implementation {#os-design-and-implementation}

There’s no known ultimate solution when it comes to OS design. Internal structures of known operating systems can vary widely.
{:.warning}

When one design an OS, it might be helpful to consider a few known things.

## User and System Goals

Start by defining <span style="color:#f77729;"><b>goals</b></span>:

1. <span style="color:#f77729;"><b>User goals</b></span>: OS should be convenient to use, easy to learn, reliable, safe, fast
2. <span style="color:#f77729;"><b>System goals</b></span>: The system should be easy to design, implement, and maintain; and it should be flexible, reliable, error free, and efficient.

## Policy and Mechanism Separation

Know the <span style="color:#f77729;"><b>difference</b></span> between <span style="color:#f7007f;"><b>policy</b></span> and <span style="color:#f7007f;"><b>mechanism</b></span> and separate them:

1. <span style="color:#f7007f;"><b>Policy</b></span>: determines what will be done
2. <span style="color:#f7007f;"><b>Mechanism</b></span>: determines how to do something

The separation of policy and mechanism is important for <span style="color:#f77729;"><b>flexibility</b></span>:

- Policies are likely to <span style="color:#f77729;"><b>change</b></span> across <span style="color:#f77729;"><b>places</b></span> or over <span style="color:#f77729;"><b>time</b></span>.
- In the worst case, each change in policy would require a change in the underlying mechanism.

A general mechanism insensitive to changes in policy would be more desirable. A change in policy would then require redefinition of only certain <span style="color:#f77729;"><b>parameters</b></span> of the system.
{:.warning}

Take for example: a <span style="color:#f77729;"><b>mechanism</b></span> for giving <span style="color:#f77729;"><b>priority</b></span> to certain types of programs over others. If the <span style="color:#f77729;"><b>mechanism</b></span> is properly <span style="color:#f77729;"><b>separated</b></span> from policy, it can be easily tweaked based on user requirements:

- <span style="color:#f77729;"><b>Support</b></span> a policy decision that I/O-intensive programs should have priority over CPU-intensive ones
- Or <span style="color:#f77729;"><b>support</b></span> the opposite policy whenever appropriate.

Either way, **no** change in the instructions need to be made.

# OS Structures

Once an operating system is designed, it must be <span style="color:#f77729;"><b>implemented</b></span>. Because operating systems are <span style="color:#f7007f;"><b>collections</b></span> of many programs: kernel, system programs, interface, etc, written by <span style="color:#f7007f;"><b>many</b></span> people over a <span style="color:#f7007f;"><b>long</b></span> period of time, it is difficult to make general statements about how they are implemented[^11].

Below are a few common OS structures.

## Monolithic Structure {#monolithic-structure}

A monolithic kernel is an operating system architecture where the entire operating system is working in kernel space. It can operate <span style="color:#f77729;"><b>with</b></span> or <span style="color:#f77729;"><b>without</b></span> dual mode.

### Without Dual Mode

The figure below (screenshot from SGG book) shows the structure of MS-DOS, one of the <span style="color:#f77729;"><b>simplest</b></span> OS made in the early years:

- The interfaces and levels of functionality are<span style="color:#f77729;"><b> not well separated</b></span> (<span style="color:#f77729;"><b>all</b></span> programs can access the hardware) - i.e: at the time, MS-DOS was written for the Intel 8088 architecture, which has no mode bit and therefore no dual mode.
- For instance, application programs are able to access the basic I/O routines to write directly to the display and disk drives.
- Such freedom leaves MS-DOS <span style="color:#f77729;"><b>vulnerable</b></span> to errant (or malicious) programs, causing entire system crashes when user programs fail.

<img src="/50005-2023/assets/images/week2/8.png"  class="center_fourty"/>

### With Dual Mode

The early UNIX OS was also simple in its form as shown below. In a way, it is layered to a <span style="color:#f77729;"><b>minimal</b></span> extent with very simple structuring.

<img src="/50005-2023/assets/images/week2/9.png"  class="center_seventy"/>

The kernel provides file system management, CPU scheduling, memory management, and other operating-system functions through system calls. That is an <span style="color:#f77729;"><b>enormous</b></span> amount of functionality to be combined into one level.

- <span style="color:#f77729;"><b>Pros</b></span>: distinct <span style="color:#f77729;"><b>performance advantage</b></span> because there is very little overhead in the system call interface or in communication within the kernel.
- <span style="color:#f77729;"><b>Cons</b></span>: difficult to implement and maintain.

Other examples of monolithic OS with dual-mode: BSD, Solaris

## Layered Approach {#layered-approach}

The operating system is broken into a many number of <span style="color:#f77729;"><b>layers</b></span> (levels). The bottom layer (layer 0) is the hardware; the highest (layer N) is the user interface. The figure below shows a layered approach (layer names for illustration purposes). The programs in layer N rely on services ONLY from the layer below it.

<img src="/50005-2023/assets/images/week2/10.png"  class="center_seventy"/>

<span style="color:#f77729;"><b>Pros</b></span>:

- <span style="color:#f77729;"><b>Simple</b></span> to construct and debug
- Each <span style="color:#f77729;"><b>layer</b></span> is implemented only with operations provided by lower-level layers.
- A layer <span style="color:#f77729;"><b>does not need to know</b></span> how these operations are implemented; it needs to know only what these operations do.
- This <span style="color:#f77729;"><b>abstracts</b></span> and hides the existence of certain data structures, operations, and hardware from higher-level layers.

<span style="color:#f77729;"><b>Cons</b></span>:

- <span style="color:#f77729;"><b>Appropriately</b></span> defining the various layers, and careful planning is necessary.
  - If we are met with <span style="color:#f77729;"><b>bugs</b></span> in our program, we debug our program and not our compiler
  - <span style="color:#f7007f;"><b>We mostly assume that the layers beneath us are already made correct</b></span>
  - Sometimes, this assumption is not always true and difficult to maintain with the growing size of the OS. Some OS is shipped with bugs on its lower layers that are very difficult for users to debug.
  - Patches and updates are periodically given to fix these bugs.
- They tend to be <span style="color:#f77729;"><b>less efficient</b></span> than other types.
  - For instance, when a process running in user mode executes an I/O operation, it executes a system call that is trapped to the I/O layer, which calls the memory-management layer, which in turn calls the CPU-scheduling layer, which is then passed to the hardware.
  - At each layer, the parameters may be modified, data may need to be passed, and so on.
  - Each layer adds overhead to the system call.
  - The net result is a system call that takes longer than does one on a non layered system.

Example: Windows NT (the later version is actually a <span style="color:#f77729;"><b>hybrid</b></span> OS, combining between layered and monolithic aspects and benefits)[^13]

<img src="/50005-2023/assets/images/week2/11.png"  class="center_fourty"/>

## Microkernel {#microkernel-structures}

A microkernel is a <span style="color:#f7007f;"><b>very small</b></span> kernel that provides <span style="color:#f77729;"><b>minimal</b></span> process and memory management, in addition to a communication facility.

This method structures the operating system by removing all nonessential components from the kernel and implementing them as system and user-level programs. The result is a smaller kernel that does only tasks pertaining to:

1. <span style="color:#f7007f;"><b>Inter-Process Communication</b></span>,
2. <span style="color:#f7007f;"><b>Memory Management</b></span>,
3. <span style="color:#f7007f;"><b>Scheduling</b></span>

For instance, if a user program wishes to access a file, it must interact with the file server.

- The client program and service never interact directly.
- Rather, they communicate indirectly by exchanging messages with the microkernel as illustrated below:

<img src="/50005-2023/assets/images/week2/12.png"  class="center_fifty"/>

<span style="color:#f77729;"><b>Pros</b></span>: extending the operating system easier. All new services are added to user space and consequently do not require modification of the kernel.

<span style="color:#f77729;"><b>Cons</b></span>: suffer in performance increased system-function overhead due to frequent requirement in performing context switch.

<span style="color:#f77729;"><b>Example</b></span>: Mach, Windows NT (first release was a microkernel).

## Hybrid Approach {#hybrid-approach}

Hybrid kernels attempt to <span style="color:#f77729;"><b>combine</b></span> between <span style="color:#f7007f;"><b>microkernel</b></span> and <span style="color:#f7007f;"><b>monolithic</b></span> kernel aspects and benefits.

<span style="color:#f77729;"><b>Example<span style="color:#f77729;"><b></b></span></b></span>: macOS is partly based on microkernel + monolithic approach (image taken from SGG):

1. Mach provides: IPC, scheduling, memory management
2. BSD provides: CLI, file system management, networking support, POSIX APIs implementations

<img src="/50005-2023/assets/images/week2/13.png"  class="center_fifty"/>

## Java Operating System (JX) {#java-operating-system-jx}

The <span style="color:#f77729;"><b>JX OS</b></span> is written almost entirely in <span style="color:#f77729;"><b>Java</b></span>. Such a system, known as a <span style="color:#f77729;"><b>language-based extensible</b></span> system, and runs in a single address space (no virtualisation, no MMU), as such it will face difficulties in maintaining memory protection that is usually supported by hardwares in typical OS.

<img src="/50005-2023/assets/images/week2/14.png"  class="center_fourty"/>

Language-based systems instead rely on <span style="color:#f77729;"><b>type-safety</b></span>[^14] features of the language. As a result, language-based systems are desirable on small hardware devices, which may lack hardware features that provide <span style="color:#f77729;"><b>memory protection</b></span>. Since Java is a type-safe language, JX is able to provide <span style="color:#f77729;"><b>isolation</b></span> between running Java applications without hardware memory protection.

This is called language based protection, where system calls and IPC in JX does not require an address-space switch. In short, JX runs in a single address space.
{:.warning}

The architecture of the JX system is illustrated below (simplified representation[^15]):

- JX organizes its system according to <span style="color:#f77729;"><b>domains</b></span>.
  <img src="/50005-2023/assets/images/week2/15.png"  class="center_seventy"/>

- Each <span style="color:#f77729;"><b>domain</b></span> represents an independent <span style="color:#f7007f;"><b>JVM</b></span> (Java Virtual Machine):
  _ JVM is an abstract virtual machine that can run on <span style="color:#f7007f;"><b>any</b></span> OS
  _ There’s one instance of JVM per Java application
  _ JVM provides portable execution environment for Java-based apps
  _ It maintains a heap used for allocating memory during object creation and threads within itself, as well as for <span style="color:#f77729;"><b>garbage collection</b></span>.
  <img src="/50005-2023/assets/images/week2/16.png"  class="center_seventy"/>

- Domain zero is a <span style="color:#f77729;"><b>microkernel</b></span> responsible for low-level details, such as system initialization and saving and restoring the state of the CPU.
  - Domain zero is written in C and assembly language; all other domains are written entirely in Java.
  - Communication between domains occurs through a specific mechanism called <span style="color:#f77729;"><b>portals</b></span>
- Protection within and between domains relies on the type safety of the Java language.
  - However, since domain zero is not written in Java, it <span style="color:#f77729;"><b>must be considered trusted</b></span> (built by trusted sources)

## Summary {#summary}

The figure below shows the summary of various OS structures.

<img src="/50005-2023/assets/images/week2/17.png"  class="center_seventy"/>

For layered architecture, note that the only difference with hybrid and microkernel structure is that programs at level N relies <span style="color:#f7007f;"><b>ONLY</b></span> on services provided by programs at <span style="color:#f77729;"><b>one</b></span> level below it.

# Appendix

If you'd like to expand your knowledge beyond regular OS, you may have further read about <span style="color:#f77729;"><b>virtualization</b></span> and <span style="color:#f77729;"><b>containerization</b></span>.

<img src="/50005-2023/assets/images/week2/18.png"  class="center_seventy"/>

## Virtualisation

We can run any arm or intel-based OS on arm or intel-based hardware, e.g: Dual boot. An extension to that is <span style="color:#f77729;"><b>virtualization</b></span>, where you can run any OS on any OS (typically the same architecture).

A <span style="color:#f77729;"><b>hypervisor</b></span> is essentially an <span style="color:#f77729;"><b>emulator</b></span> computer software applications, firmware, and hardware that runs virtual machines. Examples of hypervisors: VMWare, VMWorkstation, VirtualBox, Parallel Desktop for Mac, etc

## Containerization

<span style="color:#f77729;"><b>Containers</b></span> allow a developer to package up an application with all of the parts it needs, such as <span style="color:#f77729;"><b>libraries</b></span> and other <span style="color:#f77729;"><b>dependencies</b></span>, and ship it all out as one package. Containers are not the same as Virtual Machines, and are generally faster to use. You can read more about the differences between the two [here](https://geekflare.com/docker-vs-virtual-machine/).

Note: "Host OS" in the picture above assumes that it is UNIX-based POSIX compliant OS. If you run Docker on Windows/Mac, it will download a Hypervisor + Guest Linux too before you can run the Docker Engine.
{:.warning}

Example of software applications that support containerisation: [Docker](https://www.freecodecamp.org/news/docker-quick-start-video-tutorials-1dfc575522a0).

[^10]: `tty` itself is a command in Unix and Unix-like operating systems to print the file name of the terminal connected to standard input. If you open multiple terminal windows in your UNIX-based system and type `tty` on each of them, you will be returned with different ids. You may use write to communicate across terminal windows
[^11]: Early operating systems were written in assembly language. Now, although some operating systems are still written in assembly language, most are written in a higher-level language such as C or an even higher-level language such as C++. Actually, an operating system can be written in more than one language: (1) The lowest levels of the kernel might be assembly language, and then (2) higher-level routines might be in C, and finally (3) system programs might be in C or C++, in interpreted scripting languages like PERL or Python, or in shell scripts. In fact, a given Linux distribution probably includes programs written in all of those languages.
[^13]: Figure taken from Wikipedia
[^14]: The Java language is designed to enforce type safety. This means that programs are prevented from accessing memory in inappropriate ways. Every section of memory is part of some Java object that belongs to some class. For example, a calendar-management applet might use classes like Date, Appointment, Alarm, and GroupCalendar. Each class defines both a set of objects and operations to be performed on the objects of that class (explanation taken from Securing Java (ISBN: 047131952X), section 10)
[^15]: Image taken from https://webmobtuts.com/news-events/what-is-the-jvm-introducing-the-java-virtual-machine/
