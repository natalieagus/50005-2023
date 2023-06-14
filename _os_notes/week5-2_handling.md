---
title: Deadlock Handling Methods
permalink: /os_notes/week5_handling
key: week5_handling
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

There are three deadlock handling methods:

1. Deadlock <span style="color:#f77729;"><b>Prevention</b></span>
2. Deadlock <span style="color:#f77729;"><b>Avoidance</b></span>
3. Deadlock <span style="color:#f77729;"><b>Detection</b></span>

# Deadlock Prevention

Deadlock prevention works by <span style="color:#f77729;"><b>ensuring</b></span> that at least 1 of the necessary conditions for deadlock <span style="color:#f77729;"><b>never</b></span> happens.

Let’s examine each of the necessary conditions and decide whether we can prevent them from happening.

## Mutual Exclusion Prevention

We typically cannot prevent deadlocks by denying the mutex condition, because some resources are intrinsically non-sharable.

> For example, a mutex lock cannot be simultaneously shared by several processes.

## Hold and Wait Prevention

To ensure that hold and wait never happens, we need to guarantee that whenever a process requests for a resource, it <span style="color:#f7007f;"><b>does NOT</b></span> currently hold any other resources.

Possible through such protocols:

- <span style="color:#f77729;"><b>Protocol 1</b></span>: Must have all of its resources at once before beginning execution, otherwise, wait empty handed.
- <span style="color:#f77729;"><b>Protocol 2</b></span>: Only can request for new resources when it has none.

### Example

Consider a process that needs to `write` from a DVD drive to disk and then `print` a document from the disk to the printer machine.

<span style="color:#f77729;"><b>With protocol 1: </b></span>
The process must acquire <span style="color:#f77729;"><b>all</b></span> three resources: DVD drive, disk, and printer before beginning execution, even though the task with the disk and DVD drive has nothing to do with the printer. After it finishes both tasks, it releases both the disk and printer.

<span style="color:#f77729;"><b>With protocol 2: </b></span>
The process requests for DVD drive and disk, and writes to the disk. Then it releases <span style="color:#f77729;"><b>both</b></span> resources. Afterwards, it submits a new request to gain access to the disk (again) and the printer, prints the document, and releases both resources.

### Disadvantages

<span style="color:#f77729;"><b>Starvation</b></span>:

- Process with <span style="color:#f77729;"><b>protocol 1</b></span> _may never start_ if resources are <span style="color:#f77729;"><b>scarce</b></span> and there are too many other processes requesting for it.

<span style="color:#f77729;"><b>Low resource utilization</b></span>:

- In Protocol 1, resources are allocated at once but it is likely that they <span style="color:#f77729;"><b>aren’t used simultaneously</b></span> (e.g: the sample process requests a printer but it doesn’t utilize it when writing to disk first)
- In Protocol 2, lots of time is <span style="color:#f77729;"><b>wasted</b></span> for requesting and releasing many resources in the <span style="color:#f77729;"><b>middle</b></span> of execution.

## No Preemption Prevention (Allow Preemption)

To _allow_ preemption, we need a certain protocols in place to ensure that progress is not lost.

A process is holding some resources and requests another resource that cannot be immediately allocated to it (that is, the process must wait). We can then decide to do either protocols below:

- <span style="color:#f77729;"><b>Protocol 1</b></span>: All resources the process is currently holding are _preempted_ -- <span style="color:#f7007f;"><b>implicitly released</b></span>.
- <span style="color:#f77729;"><b>Protocol 2</b></span>: Check first if the resources requested are held by other waiting processes or not.
  - If held by other <span style="color:#f77729;"><b>waiting</b></span> process: preempts the waiting process and give the resource to the requesting process
  - If neither held by waiting process nor available: requesting process must wait.

This protocol is often applied to resources whose state can be easily <span style="color:#f77729;"><b>saved</b></span> and <span style="color:#f77729;"><b>restored</b></span> later, such as CPU registers and memory space. It cannot generally be applied to such resources as mutex locks and semaphores.
{:.warning}

## Circular Wait Prevention

To prevent circular wait, one must have a protocol that impose a <span style="color:#f7007f;"><b>total ordering of all resource types</b></span>, and require that each process requests resources _according_ to that order.
{:.warning}

### Example

We can assign some kind of `id` to each resource and we need to design some acquiring protocols such as:

- The <span style="color:#f77729;"><b>highest</b></span> priority order of currently-held resources should be _less than or equal to_ the current resource being requested, otherwise the process must release the resources that violate this condition.
- Each process can request resources only in an <span style="color:#f77729;"><b>increasing</b></span> order of enumeration.

> This burdens the programmer to ensure the order by desig to ensure that it doesn't sacrifice resource utilization unnecessarily.

# Deadlock Avoidance

In deadlock <span style="color:#f7007f;"><b>avoidance</b></span> solution, we need to spend some time to perform an algorithm to <span style="color:#f7007f;"><b>check</b></span> <span style="color:#f7007f;"><b>BEFORE</b></span> granting a resource request, _even if the request is valid and the requested resources are now available_.
{:.warning}

This Deadlock Avoidance algorithm is called the Banker's Algorithm. It's job is to <span style="color:#f77729;"><b>compute</b></span> whether or not the current request _will_ lead to a deadlock.

- If yes, the request for the resources will be rejected,
- If no, the request will be granted.

This algorithm is run <span style="color:#f7007f;"><b>each time</b></span> a process request the shared resources.

# The Banker's Algorithm

The Banker's Algorithm is comprised of <span style="color:#f77729;"><b>two</b></span> parts: The Safety Algorithm and the Resource Allocation Algorithm. The latter utilises the output of the former to determine whether the currently requested resource should be granted or not.

## Required Attributes

Before we can run the algorithm, we need to know:

1. Number of processes (consumers) in the system (denoted as `N`)
2. Number of resource <span style="color:#f77729;"><b>types</b></span> in the system (denoted as `M`), along with _initial instances_ of each resource type at the start.
3. The <span style="color:#f77729;"><b>maximum</b></span> number of resources required by each process (consumers).

## The System State

The algorithm maintains these four data structures representing the <span style="color:#f7007f;"><b>system STATE</b></span>:

1. available: 1 by M vector

   - `available[i]`: the available instances of resource `i`

2. max: N by M matrix

   - `max[i][j]`: maximum demand of process `i` for resource `j` instances

3. allocation: N by M matrix

   - `allocation[i][j]`: current allocation of resource `j` instances for process `i`

4. need: N by M matrix
   - `need[i][j]`: how much more of resource `j` instances might be needed by process `i`

## Part 1: Resource Allocation Algorithm

You shall read about this algorithm during Lab. As such, the explanation of the algorithm will not be repeated here.
{:.warning}

As time progresses, more <span style="color:#f77729;"><b>resource request</b></span> or <span style="color:#f77729;"><b>resource release</b></span> can be invoked by any of the processes.

- <span style="color:#f77729;"><b>Releasing</b></span> Resources is trivial, we simply update the `allocation`, `available`, and `need` data structures.
- For each Resource Request received, the output of the algorithm can be either `Granted` or `Rejected` depending on whether the system will be in a <span style="color:#f77729;"><b>safe state</b></span> _if the resource is granted_

### System State Update

Note that these requests are made sequentially in time, so don’t forget to update the system state as you grant each request. When considering subsequent new requests, we perform the resource allocation algorithm with the <span style="color:#f77729;"><b>UPDATED</b></span> states that’s modified if you have granted the previous request.

If the previous request is rejected however, no change in system state is made and you can leave it as is.

## Part 2: The Safety Algorithm

This algorithm receives a <span style="color:#f7007f;"><b>copy</b></span> of `available` (named as `work`), `need`, and `allocation` data structures to perform a _hypothetical situation_ of whether the system <span style="color:#f7007f;"><b>will</b></span> be in a safe state if the current request is granted.

If we find that all processes <span style="color:#f77729;"><b>can still finish</b></span> their task (the elements of the `finish` vector is all `True`), then the system is in a safe state and we can grant this current request.

- If you store each value of index `i` acquired, you can compute the sequence of _possible_ process execution sequence

For example, given the following system state:
<img src="/50005/assets/images/week5/4.png"  class="center_seventy"/>

and current request `R_1` made by `P1`: `[1,0,2]`, you may find that granting this request leads to a <span style="color:#f7007f;"><b>safe state</b></span> and there exist two possible execution sequence (depending on _how_ you iterate through the `finish` vector (from index 0 onwards or from index `N-1` backwards):

1. `P1, P3, P4, P0, P2`, or
2. `P1, P3, P4, P2, P0`

## Complexity

Deadlock avoidance is <span style="color:#f77729;"><b>time-consuming</b></span>, since due to the expensive `while` loop in the safety algorithm. The time complexity of the safety algorithm is $$O(MN^2)$$ (which is also the complexity of the Banker's Algorithm, since the complexity of the Resource Allocation Algorithm is way smaller).

Carefully think: _why_. Also, ask yourself: what is the space complexity of the Banker's Algorithm?
{:.warning}

# Deadlock Detection

In deadlock detection, we <span style="color:#f7007f;"><b>allow</b></span> deadlock to happen first (we don't deny its possibility), and then detect it later.

It is different from deadlock avoidance. The latter is one step ahead since will not grant requests that may lead to deadlocks in the first place.
{:.warning}

Deadlock detection mechanism works by providing two things:

- An algorithm that <span style="color:#f77729;"><b>examines</b></span> the state of the system to determine whether a deadlock <span style="color:#f7007f;"><b>has occurred</b></span>
- An algorithm to <span style="color:#f77729;"><b>recover</b></span> from deadlock condition

## Deadlock Detection Algorithm

This algorithm works similarly Part 2 of the Banker’s Algorithm (the safety algorithm), just that this algorithm is performed on the _actual_ system state (not the hypothetical `work`, `need` and `allocation`!)

Firstly, we need all the state information as in Banker’s algorithm: the `available`, `request`(renamed from `need`), and `allocation` matrices (no max/need). The only subtle difference is only that the `need` matrix (in safety algorithm) is replaced by the `request` matrix.

In a system that adapt deadlock <span style="color:#f77729;"><b>detection</b></span> as a solution to the deadlock problem, we don’t have to know the `max`(resource needed by a process), because we will _always_ grant a request whenever whatever’s requested is <span style="color:#f77729;"><b>available</b></span>, and invoke deadlock detection algorithm from time to time to ensure that the system is a good state and not deadlocked.
{:.warning}

The `request` matrix contains current requests of resources made by <span style="color:#f77729;"><b>all</b></span> processes at the instance we decide to <span style="color:#f7007f;"><b>detect</b></span> whether or not there’s (already) a deadlock occuring in the system.

- Row `i` in the `request` matrix stands for: Process `i` requiring some <span style="color:#f77729;"><b>MORE</b></span> resources that’s what’s been allocated for it.

The deadlock detection algorithm goes as follows:

- <span style="color:#f77729;"><b>Step 1</b></span>: Initialize these two vectors:

  - `work` (length is #resources `M`) initialized to be == `available`
  - `finish` (length is #processes `N`) initialized to be:
    - `False` if `request[i]` != {0} (empty set)
    - True `otherwise` (means process i doesn’t request for anything else anymore and can be resolved or finished)
  - No update of `allocation, need` needed. We are checking whether the CURRENT state is safe, not whether a HYPOTHETICAL state is safe.

- <span style="color:#f77729;"><b>Step 2</b></span>: find an index `i` such that <span style="color:#f77729;"><b>both</b></span> conditions below are fulfilled,

  - `Finish[i] == False`
  - `request[i] <= work` (element-wise comparison for these vectors)

- <span style="color:#f77729;"><b>Step 3</b></span>:
  - If Step 2 produces such index `i`, update `work` and `finish[i]`,
    - `work += allocation[i]` (vector addition)
    - `finish[i] = True`
    - <span style="color:#f7007f;"><b>Go back to Step 2</b></span>
  - If Step 2 does not produce such index `i`, prepare for exit:
    - If `finish[i] == False` for some `i`, then the system is <span style="color:#f7007f;"><b>already</b></span> in a deadlock (caused by Process `i` getting deadlocked by another Process `j` where `finish[j] == False` too)
    - Else if `finish[i] == True` for all `i<N`, then the system is <span style="color:#f7007f;"><b>not in a deadlock</b></span>.

## Complexity

This deadlock detection algorithm requires an order of $$O(MN^2)$$ <span style="color:#f77729;"><b>operations</b></span> to perform.

## Frequency of Detection

When should we invoke the detection algorithm? The answer depends on two factors:

- How often is a deadlock likely to occur?
- How many processes will be affected by deadlock when it happens?

This remains an open issue.

# Deadlock Recovery

To recover from deadlock, we can either:

1. Abort <span style="color:#f77729;"><b>all</b></span> deadlocked processes, the resources held are preempted
2. Abort deadlocked processes <span style="color:#f77729;"><b>one at a time</b></span>, until deadlock no longer occurs

The second method results in <span style="color:#f7007f;"><b>overhead</b></span> since we need to run the detection algorithm over and over again each time we abort a process. There are a few design choices:

- Which <span style="color:#f77729;"><b>order</b></span> shall we abort the processes in?
- Which “victim” shall we select? We need to determine <span style="color:#f77729;"><b>cost</b></span> factors such as which processes have executed the longest, etc
- If we select a “victim”, how much <span style="color:#f77729;"><b>rollback</b></span> should we do to the process?
- How do we ensure that <span style="color:#f77729;"><b>starvation</b></span> does not occur?

Deadlock Recovery remains an open ended issue. Dealing with deadlock is also a difficult problem. Most operating systems do not prevent or resolve deadlock completely and users will deal with it when the need arises.
{:.warning}
