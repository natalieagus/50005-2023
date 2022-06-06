---
title: Critical Section Solutions
permalink: /os_notes/week4_solutions
key: os-notes-week4_solutions
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
In the next few sections, we discuss several known solutions to CS problems. They are generally divided into these categories:
* <span style="color:#f77729;"><b>Software Mutex </b></span>Algorithm (purely software only, possible only under restricted conditions, busy waits)
* <span style="color:#f77729;"><b>Hardware</b></span> Supported <span style="color:#f77729;"><b>Spinlocks</b></span> (hardware supported, only 1 process in the CS at a time, busy waits)
* <span style="color:#f77729;"><b>Semaphores</b></span> (does not busy wait, generalization of mutex locks -- able to protect two or more identical shared resources) 
* <span style="color:#f77729;"><b>Conditional Variables</b></span> (does not busy wait, wakes up on condition) 

# Software Mutex Algorithm
## Peterson'S=s Solution
Peterson’s solution is a <span style="color:#f77729;"><b>software</b></span>-based approach that solves the CS problem, but two restrictions apply:
* Strictly <span style="color:#f77729;"><b>two</b></span> processes that <span style="color:#f77729;"><b>alternate</b></span> execution between their critical sections and remainder sections (can be generalised to `N` processes with proper data structures, out of syllabus)
* Architectures where `LD` and `ST` are <span style="color:#f77729;"><b>atomic</b></span>[^1] (i.e: executed in 1 clk cycle, or not interruptible)


The solution works by utilizing two shared global variables:
```cpp
int turn;
bool flag[2];
```
* If `turn == i`, process `Pi` is allowed to <span style="color:#f77729;"><b>enter</b></span> the critical section. Similar otherwise.
* If `flag[i] == true`, then process `Pi` is <span style="color:#f77729;"><b>ready</b></span> to enter the critical section.

### Initialisation
We can initialize the `flag` into `false` for all `i`, and set `turn` into <span style="color:#f77729;"><b>arbitrary number</b></span> (`i` or `j` to index two processes `Pj` and `Pi` -- two processes only for our syllabus) in the beginning.

### Algorithm
```cpp
do{
   flag[i] = true;
   turn = j;
   while (flag[j] && turn == j); // this is a while LINE
   // CRITICAL SECTION HERE
   // ...
   flag[j] = false;
   // REMAINDER SECTION HERE
   // ...
}while(true)
```

The algorithm above is the solution for process `Pi`. 
* In the `while`-loop, `Pi` busy waits (means <span style="color:#f77729;"><b>try</b></span> and keep <span style="color:#f77729;"><b>retrying</b></span> until succeed, thus wasting the quanta given to the process), 
* `Pi` will be <span style="color:#f7007f;"><b>stuck</b></span> at the while-line (notice it's NOT a while-loop, there’s a semicolon at the end), for as long as `flag[j] == true` <span style="color:#f7007f;"><b>and</b></span> `turn == j`. 

## Prove of Correctness
The prove that this solution is correct, we need to show that:
* <span style="color:#f77729;"><b>Mutual exclusion</b></span> is preserved.
* The <span style="color:#f77729;"><b>progress</b></span> requirement is satisfied.
* The <span style="color:#f77729;"><b>bounded-waiting</b></span> requirement is met.


We prove that the Peterson’s solution satisfies all <span style="color:#f77729;"><b>three</b></span> properties above by tracing how it works:
* When `Pi` wants to enter the critical section, it will set its own flag; `flag[i]` into `true`. 
* Then, it sets `turn=j` (instead of itself, i). 
> You can think of a process as being <span style="color:#f77729;"><b>polite</b></span> hence requesting the *other one* (`j`) to be executed <span style="color:#f77729;"><b>first</b></span>.

Now two different scenarios might happen.

### Scenario 1: Proceed to CS
`Pi`might break from the `while`-loop under two possible conditions.

<span style="color:#f77729;"><b>Conditon 1:</b></span> `flag[j] == false`, meaning that the other `Pj`` is <span style="color:#f7007f;"><b>not ready</b></span> to enter the CS and is <span style="color:#f7007f;"><b>also not in the critical section</b></span>. This ensures <span style="color:#f77729;"><b>mutex</b></span>.

<span style="color:#f77729;"><b>Condition 2:</b></span> OR, IF `flag[j] == true` but `turn == i`. This means the other process `Pj` is also <span style="color:#f77729;"><b>about</b></span> to enter the critical section. No process is in the Critical Section, but it is `Pi`'s turn, so `Pi` gets to enter the CS first (ensuring <span style="color:#f77729;"><b>progress</b></span>). 

### Scenario 2: Busy-wait
`Pi` will be stuck with the `while`-line if `flag[j] == true` <span style="color:#f7007f;"><b>AND</b></span> `turn == j`, meaning that `Pj`is inside the CS. 

This satisfies <span style="color:#f77729;"><b>mutual exclusion</b></span>: `flag[i]` and `flag[j]` are both `true`, but `turn` is an <span style="color:#f77729;"><b>integer</b></span> and cannot be both `i` and `j`. 
* No two processes can simultaneously execute the CS. 

<span style="color:#f77729;"><b>Bounded waiting is guaranteed</b></span>: After `Pj` is done, it will set its own flag: `flag[j] = false`, hence ensuring `Pi` to <span style="color:#f77729;"><b>break</b></span> out of the `while`-line and enter CS in the future. 
* `Pi` only needs to wait a <span style="color:#f7007f;"><b>MAXIMUM</b></span> of 1 cycle before being able to enter the CS.

> You might want to <span style="color:#f77729;"><b>interleave</b></span> the execution of instructions between `Pi` and `Pj`, and convince yourself that this solution is indeed a legitimate solution to the CS problem. Try to also modify some parts: not to use `turn`, set `turn` for `Pi` as itself (`i`) instead of `j`, not to use `flag`, etc and convince yourself whether the 3 <span style="color:#f77729;"><b>correctness</b></span> property for the original Peterson's algorithm still apply. 



[^1]:
   An operation acting on shared memory is atomic  if it completes in a single step relative to other threads. For example, when an atomic store is performed on a shared variable, no other thread/process can observe the modification half-complete.  