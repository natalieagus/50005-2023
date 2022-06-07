---
title: Operations on Processes
permalink: /os_notes/week3_operations
key: os-notes-week3_operations
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

We can perform various <span style="color:#f77729;"><b>operations</b></span> on a process: spawning child processes, terminate the process, set up inter-process communication channels, change the process priority, and many more. All of these operations require a system call (switching to Kernel Mode). In this example, we use the <span style="color:#f77729;"><b>C API</b></span> to make the system call. 

# Process Creation  {#process-creation}

We can create new processes using <code>fork()</code> system call.
1. The process creator is called a <span style="color:#f7007f;"><b>parent</b></span> process, the new processes are called the <span style="color:#f7007f;"><b>children</b></span> of that process. 
2. Each of these new processes may in turn create more child processes, forming a <span style="color:#f7007f;"><b>tree</b></span> of processes.




## Process Tree
We can illustrate multiple process creation as a <span style="color:#f77729;"><b>process tree</b></span>:

<img src="/50005/assets/images/week3/8.png"  class="center_fourty"/>

In the example above, there are 5 processes in total. Process 2,3, and 4 are direct <span style="color:#f77729;"><b>children</b></span> of Process 1. Process 5 is created by Process 2. 



### Process id
Each process is identified by an integer called the process id (`pid`). Pid is <span style="color:#f7007f;"><b>unique</b></span> in the system.
{:.warning}

You can type the command `ps [options]` to observe all running processes in your system, along with the `pid` of each process. For instance,

<img src="/50005/assets/images/week3/9.png"  class="center_fifty"/>

### Child Process vs Parent Process

The <span style="color:#f77729;"><b>new</b></span> process consists of the entire copy of the address space (code, stack, process of execution, etc) of the <span style="color:#f77729;"><b>original</b></span> parent process at the point of `fork()`. 

> In other words, the child process inherits the parent process' state <span style="color:#f7007f;"><b>at the point of</b></span> `fork()`. 

Parent and child processes operate in <span style="color:#f77729;"><b>different address space</b></span> (isolation). Since they are different processes, parent and children processes execute <span style="color:#f77729;"><b>concurrently</b></span>.

Practically, a parent process <span style="color:#f77729;"><b>waits</b></span> for its children to terminate (using `wait()` system call) to read the child process’ exit status and <span style="color:#f77729;"><b>only then</b></span> its PCB entry in the process table can be removed. 

Child processes <span style="color:#f77729;"><b>cannot</b></span> `wait` for their parents to terminate. Since children processes are a duplicate of their parents (inherits the whole address space), they can either
 1. <span style="color:#f77729;"><b>Execute</b></span> the same instructions as their parents concurrently, or 
 2. <span style="color:#f77729;"><b>Load</b></span> a new program into its address space 


## Program: How fork works {#code-how-fork-works}

It is best to explain how `fork()` process creation works by example. 


```java
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
   pid_t pid;

   pid = fork();
   printf("pid: %d\n", pid);

   if (pid < 0)
   {
       fprintf(stderr, "Fork has failed. Exiting now");
       return 1; // exit error
   }
   else if (pid == 0)
   {
       execlp("/bin/ls", "ls", NULL);
   }
   else
   {
       wait(NULL);
       printf("Child has exited.\n");
   }
   return 0;
}
```


The simple C program above is executed and when the execution system call `fork()` returns, <span style="color:#f77729;"><b>two processes are present</b></span>.

<img src="/50005/assets/images/week3/10.png"  class="center_seventy"/>

Both have the <span style="color:#f77729;"><b>same</b></span> copy of the text (code) and resources (any opened files, etc). The parent process is <span style="color:#f77729;"><b>cloned</b></span>, resulting in the child process. They're at a <span style="color:#f77729;"><b>different</b></span> address space, executed concurrently by the system. 

### fork return value
<code>fork()</code>returns 0 in the child process while in the parent process it returns the pid of the child (>0).
{:.warning}

We can write just <span style="color:#f77729;"><b>one instruction</b></span> for <span style="color:#f77729;"><b>both parent and child process</b></span> but each will take a different <span style="color:#f77729;"><b>branch</b></span> of the `if` clause. In the example code above:
* The child executes the line if-clause: `execlp` 
* The parent process executes the `else` clause where it `wait` for the child process to `exit` 

<img src="/50005/assets/images/week3/11.png"  class="center_seventy"/>


### execlp

`execlp` is a  system call that loads a new program called `ls` onto the child process’ address space, effectively <span style="color:#f7007f;"><b>replacing</b></span> its text (code), data, and stack content. 

### wait
Concurrently, the parent process executes `wait(NULL)`, which is a system call that <span style="color:#f7007f;"><b>suspends</b></span> the parents’ execution until this child process that is executing `ls `has returned.


## Program: The fork tree {#code-the-fork-tree}

<span style="color:#f77729;"><b>Compile</b></span> and <span style="color:#f77729;"><b>run</b></span> the C program below. 

How many processes are created in total? (excluding the parent process). Can you draw the process tree? 
{:.info}


```cpp
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
   int level = 3;
   pid_t pid[level];

   for (int i = 0; i < level; i++)
   {
       pid[i] = fork();

       if (pid[i] < 0)
       {
           fprintf(stderr, "Fork has failed. Exiting now");
           return 1; // exit error
       }
       else if (pid[i] == 0)
       {
           printf("Hello from child %d \n", i);
       }
   }

   return 0;
}
```

## Process Termination {#process-termination}
A process needs certain resources (CPU time, memory, files, I/O devices) to run and accomplish its task. These resources are <span style="color:#f77729;"><b>limited</b></span>.  

A process can terminate <span style="color:#f7007f;"><b>itself</b></span> using <code>exit()</code> system call. A process can also terminate <span style="color:#f77729;"><b>other processes</b></span> using the `kill(pid, SIGKILL)` system call. 
{:.warning}

Once a process is terminated, these resources are <span style="color:#f77729;"><b>freed</b></span> by the kernel for <span style="color:#f77729;"><b>other</b></span> processes. Parent processes may <span style="color:#f7007f;"><b>terminate</b></span> or <span style="color:#f7007f;"><b>abort</b></span> its children as it knows the `pid` of its children. 

### Orphaned processes
If a parent process with live children is terminated, the children processes become <span style="color:#f7007f;"><b>orphaned</b></span> processes:
* Some operating system is designed to either <span style="color:#f7007f;"><b>abort</b></span> all of its orphaned children (cascading termination) or
* <span style="color:#f7007f;"><b>Adopt</b></span> the orphaned children processes (`init` process usually will adopt orphaned processes)


#### About `init`
In UNIX-like OS, `init` is the first process started by the kernel during booting of the computer system. `Init` is a <span style="color:#f7007f;"><b>daemon</b></span> process that continues running until the system is shut down (see Appendix). It is the direct or indirect <span style="color:#f7007f;"><b>ancestor</b></span> of all other processes, and automatically adopts all orphaned processes. 

`Init` is a <span style="color:#f7007f;"><b>user</b></span> process like any other processes, and hence it is using virtual memory. The only special thing about `init` is that it is one of the two processes that the kernel started initially. When `init` is started by the kernel, it goes into user mode. When `init` calls system call `fork(),` it traps into the kernel mode, and the kernel does certain things to _create the new process,_ and the new process _will be scheduled in the future._ When the `fork() `returns, the original process is back to user mode. The equivalent of `init` in macOS is [`launchd`](https://en.wikipedia.org/wiki/Launchd). 


### Zombie Processes {#zombie-processes}

<span style="color:#f7007f;"><b>Zombie</b></span> processes are processes that are <span style="color:#f7007f;"><b>ALREADY TERMINATED</b></span>, and memory as well as other resources are <span style="color:#f7007f;"><b>freed</b></span>, but its exit status is not read by their parents, hence its <span style="color:#f7007f;"><b>entry</b></span> (PCB) in the process table remains. 
{:.warning}

A parent process <span style="color:#f7007f;"><b>must</b></span> call `wait` or `waitpid` to read their children’s exit status. A call to `wait` or `waitpid` <span style="color:#f7007f;"><b>blocks</b></span> the calling process until one of its child processes exits or a signal is received. <span style="color:#f7007f;"><b>Otherwise, their child process becomes a zombie process. </b></span>
* Children processes can <span style="color:#f77729;"><b>terminate</b></span> themselves after they have finished executing their tasks using `exit(int status)` system call. 
* The kernel will <span style="color:#f77729;"><b>free</b></span> the memory and other resources from this process, <span style="color:#f7007f;"><b>but not the PCB entry</b></span>. 
* Parent processes are supposed to call `wait` or `waitpid` to obtain the exit status of a child.
* Only after `wait` or `waitpid` in the parent process returns, the kernel can <span style="color:#f7007f;"><b>remove</b></span> the child PCB entry from the system wide process table. 
* If the parents didn’t call `wait` or `waitpid` and instead continue execution of other things, then children’s entry in the pcb remains; <span style="color:#f7007f;"><b>a zombie process remains</b></span>. 
  * A zombie process generally takes up very little memory space, but `pid` of the child remains 
  * Recall that pid is <span style="color:#f7007f;"><b>unique</b></span>, hence in a 32-bit system, there're only 32768 available pids. 

Having too many zombie processes might result in inability to create new processes in the system, simply because we may run out of pid. 
{:.warning}

> Note: if a parent process has died as well, then all the zombie children will be <span style="color:#f7007f;"><b>cleared</b></span> by the kernel. To <span style="color:#f7007f;"><b>observe</b></span> zombie children, you need to artifically suspend the parent process after the children have terminated. 


All processes transition to this zombie state when they terminate, but generally they exist as zombies only <span style="color:#f7007f;"><b>briefly</b></span>. Once the parent calls <code>wait, waitpid</code> the `pid` of the zombie process and its entry in the process table are released.


## Program: Zombie making {#code-zombie-making}

<span style="color:#f77729;"><b>Compile</b></span> and <span style="color:#f77729;"><b>run</b></span> the C program below. It will suspend itself at `scanf`, waiting for input at `stdin`. Do not type anything, leave it hanging there. 

What’s the (possible) maximum number of zombies created by this process? 
{:.info}

```cpp
#include <sys/wait.h>
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    int level = 5;
    pid_t pid[level];

    for (int i = 0; i < level; i++)
    {
        pid[i] = fork();

        if (pid[i] < 0)
        {
            fprintf(stderr, "Fork has failed. Exiting now");
            return 1; // exit error
        }
        else if (pid[i] == 0)
        {
            printf("Hello from child %d \n", i);
            return 0;
        }
    }

    int testInteger;
    printf("Enter an integer: "); // artificially blocking parent
    scanf("%d", &testInteger);

    return 0;
}
```

We can enter the `ps aux | grep 'Z'` command to list all zombie processes in the system caused by running the program above. 
<img src="/50005/assets/images/week3/12.png"  class="center_seventy"/>
