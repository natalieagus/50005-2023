---
title: Process and Thread Synchronization
permalink: /os_notes/week4_synchronization
key: os-notes-week4_synchronization
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

## The Producer Consumer Problem

Consider the basic producer-consumer problem, where two <span style="color:#f77729;"><b>asynchronous</b></span> processes or threads are trying to write to and read from the same bounded N-character buffer <span style="color:#f77729;"><b>concurrently</b></span>.

Asynchronous and concurrent: both processes or threads make progress, but we cannot assume anything about their relative speed of execution.
{:.warning}

<img src="/50005/assets/images/week4/1.png"  class="center_seventy"/>

Confused about the term **asynchronous** or **concurrent** or **parallel**? Read [this burger ordering analogy](https://fastapi.tiangolo.com/async/) that was found by one of your CSD seniors.
{:.highlight}

### Real Life Examples

In real life situations, a producer process produces information that is consumed by a consumer process. For example:

- A <span style="color:#f77729;"><b>compiler</b></span> (producer) producing assembly code that is consumed by an <span style="color:#f77729;"><b>assembler</b></span> (consumer).
- The assembler can also be a producer with respect to the <span style="color:#f77729;"><b>loader</b></span>: it produces object modules that are consumed by the loader.
- A web <span style="color:#f77729;"><b>server</b></span> produces (that is, provides) HTML files and images, which are consumed (that is, read) by the client web <span style="color:#f77729;"><b>browser</b></span> requesting the resource.

### Precedence Constraints

The issue is that we require the following precedence constraints for these asynchronous processes:

- <span style="color:#f7007f;"><b>Producer cannot produce too much before consumer consumes </b></span>
- <span style="color:#f7007f;"><b>Consumer cannot consume before producer produces</b></span>

### No Sync

To highlight what happens if we don’t synchronize both processes / threads, lets see a very simple <span style="color:#f77729;"><b>producer</b></span> program. Let `counter` and `buffer` of size `N` be a <span style="color:#f77729;"><b>shared</b></span> variable between the producer and consumer. `counter` keeps track of how many items there are in the `buffer`.

```cpp
while (true) {
    /* produce an item in next produced */
    while (counter == BUFFERSIZE); /* do nothing */
    buffer[in] = nextproduced;
    in = (in + 1) % BUFFERSIZE;
    counter++;
}
```

And the following very simple <span style="color:#f77729;"><b>consumer</b></span> program:

```cpp
while (true) {
    while (counter == 0); /* do nothing */
    next consumed = buffer[out];
    out = (out + 1) % BUFFERSIZE;
    counter--;
    /* consume the item in next consumed */
}
```

Each code is correct on its own, but incorrect when executed <span style="color:#f7007f;"><b>concurrently</b></span>, meaning that without any proper synchronisation attempts, the <span style="color:#f7007f;"><b>precedence</b></span> condition will be <span style="color:#f7007f;"><b>violated</b></span>.
{:.error}

This is due to the presence of <span style="color:#f7007f;"><b>race condition</b></span>.

# The Race Condition

Assume `buffer` and `counter` are shared between the two processes / threads. The instructions `counter ++` and `counter -- `are not implemented in a single clock cycle (it is <span style="color:#f77729;"><b>not atomic</b></span>).

## Non-atomic ++ and --

`counter++` may be implemented as follows in assembly language:

```armasm
LDR(counter, R2)
ADDC(R2, 1, R2) || or SUBC for counter--
ST(R2, counter)
```

## Race Condition Outcome 1

The execution between `counter ++ `and `counter --` can therefore be <span style="color:#f7007f;"><b>interleaved</b></span>. For example in a uniprocessor system, when the value of `counter` is `4` and the producer is now writing the 5th item, it could get interrupted while executing `counter++` for consumer to consume the 5th item and cause the following interleaved execution between the two:

```armasm
LDR(counter, R2) | Producer executes, then interrupted, R2’s content:4
...IRQ on producer, save state, restore consumer
LDR(counter, R2) | Consumer executes, R2 contains 4
SUBC(R2, 1, R2)  | R2 contains 3, then consumer is interrupted
...IRQ on consumer, save state, restore producer
ADDC(R2, 1, R2)  | R2 contains 5
ST(R2, counter)  | value 5 is stored at counter, then IRQ
...IRQ on producer, save state, restore consumer
ST(R2, counter)  | value 3 is stored at counter
```

Therefore the final value of the `counter` is `3` when it should be `4`.

## Race Condition Outcome 2

We can try another combination of interleaved execution:

```armasm
LDR(counter, R2) | Producer executes, then interrupted R2’s content:4
...IRQ on producer, save state, restore consumer
LDR(counter, R2) | Consumer executes, R2 contains 4
SUBC(R2, 1, R2)  | R2 contains 3
ST(R2, counter)  | value 3 is stored at counter, then consumer is interrupted
...IRQ on consumer, save state, restore producer
ADDC(R2, 1, R2)  | R2 contains 5
ST(R2, counter)  | value 5 is stored at counter
```

Therefore in the case above, the final value of the `counter` is `5` which is still incorrect.

You may easily find that the value of the counter can be 4 as well through other combinations of interleaved execution of `counter ++` and `counter --`.

Hence the value of the counter <span style="color:#f7007f;"><b>depends on the order of execution</b></span> that is out of the user's control (the order of execution depends on the kernel’s scheduling handler, unknown to the user).

> In such a situation <span style="color:#f7007f;"><b>where several processes access and manipulate the same data concurrently and the outcome of the execution depends on the particular order in which the access takes place</b></span>, is called a <span style="color:#f7007f;"><b>race condition</b></span>.

A race condition is <span style="color:#f7007f;"><b>NOT desirable</b></span>. We need to perform process <span style="color:#f7007f;"><b>synchronization</b></span> and <span style="color:#f7007f;"><b>coordination</b></span> among cooperating processes (or equivalently, threads cooperation) to avoid the race condition.

# The Critical Section Problem

We define the <span style="color:#f7007f;"><b>regions</b></span> in a program whereby atomicity must be guaranteed as the <span style="color:#f7007f;"><b>critical section</b></span>.
{:.error}

Consider a system consisting of `n` processes `{P0, P1,..., Pn−1}`. Each process may have a <span style="color:#f77729;"><b>segment</b></span> of instructions, called a `critical section` (CS). The important <span style="color:#f7007f;"><b>feature</b></span> of the system is that <span style="color:#f77729;"><b>when one process is executing its critical section, no other process is allowed to execute its critical section</b></span>.

In the consumer producer sample code above, the critical section in the producer’s code is the instruction `counter++` while the critical section in the consumer’s code is `counter-—`.

In the critical section the (asynchronous) processes may be:

- Changing <span style="color:#f77729;"><b>common</b></span> variables,
- Updating a <span style="color:#f77729;"><b>shared</b></span> table,
- Writing to a <span style="color:#f77729;"><b>common</b></span> file, and so on.

To support the CS, we need to design a <span style="color:#f77729;"><b>protocol</b></span> that the processes can use to cooperate or synchronize.

> It is a challenging problem to <span style="color:#f77729;"><b>guarantee</b></span> a critical section. Therefore, having a critical section in your program is a <span style="color:#f77729;"><b>problem</b></span>, and it requires complex synchronisation solutions.

There are <span style="color:#f77729;"><b>two</b></span> basic forms of synchronization:

- <span style="color:#f77729;"><b>The mutual exclusion</b></span>: No other processes can execute the critical section if there is already one process executing it.
- <span style="color:#f77729;"><b>Condition synchronization</b></span>: Synchronize the execution of a process in a CS based on certain conditions instead.

## Requirements for CS Solution

A solution to guarantee a critical-section must satisfy the following three requirements:

- <span style="color:#f77729;"><b>Mutual exclusion</b></span> (mutex): No other processes can execute the critical section if there is already one process executing it (in the case of condition synchronization, this is adjusted accordingly)
- <span style="color:#f77729;"><b>Progress</b></span>: If there’s no process in the critical section, and some other processes wish to enter, we need to grant this permission and we cannot postpone the permission indefinitely.
- <span style="color:#f77729;"><b>Bounded waiting</b></span>: If process A has requested to enter the CS, there exists a bound on the number of times other processes are allowed to enter the CS before A. This implies that CS is also of a finite length, it cannot loop forever and will exit after a finite number of instructions.

## Properties

The requirements above result in the following property to a CS solution:

- <span style="color:#f77729;"><b>Safety</b></span> property: no race condition
- <span style="color:#f77729;"><b>Liveness</b></span> property: a program with proper CS solution will not hang forever (because technically no progress IS mutex).

## Solution Template

The solution template to a CS problem is as follows:

```cpp
while(true){

   [ENTRY SECTION]
      CRITICAL SECTION ...
      ...
   [EXIT SECTION]
      REMAINDER SECTION ...
      ...
}
```

The protocol to approach a CS in general causes the process to:

- <span style="color:#f77729;"><b>Request</b></span> for permission to enter the section (entry section).
- <span style="color:#f77729;"><b>Execute</b></span> the critical section when the request is granted
- <span style="color:#f77729;"><b>Exit </b></span> the CS solution
- There may exist an _remainder_ section

The rest of the program that is not part of the critical section is called the <span style="color:#f77729;"><b>remainder</b></span> section.
{:.warning}
