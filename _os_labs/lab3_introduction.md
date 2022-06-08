---
title: Banker's Algorithm
permalink: /os_labs/lab3_intro
key: lab3_intro
license: false
layout: article
nav_key: os_labs
sidebar:
  nav: os_labs
license: false
aside:
  toc: true
show_edit_on_github: false
show_date: false
---

In <span style="color:#f7007f;"><b>deadlock avoidance</b></span>, before granting a resource request (even if the request is valid and the requested resources are now available), we need to <span style="color:#f77729;"><b>check</b></span> that the request will not cause the system to enter a deadlock state in the future or immediately. 

The Banker's Algorithm is a resource allocation and deadlock avoidance algorithm. It can be used to predict and compute whether or not the current request will lead to a <span style="color:#f77729;"><b>deadlock</b></span>.
* If yes, the request for the resources will be <span style="color:#f77729;"><b>rejected</b></span>, 
* Otherwise, it will be <span style="color:#f77729;"><b>granted</b></span>. 

There are two parts of the Banker's Algorithm:
1. <span style="color:#f77729;"><b>Resource Allocation</b></span> Algorithm
2. <span style="color:#f77729;"><b>Safety</b></span> Algorithm 

You are required to complete the <span style="color:#f77729;"><b>Safety Algorithm</b></span> in this lab. You need Python 3.x to complete this lab. 

## Starter Code 

### Install Python 3.10.x or above
If you don't have Python 3.10.x installed already, here's a list of commands you can enter to install:

```bash
sudo apt update && sudo apt upgrade
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.10 -y
```

When done, check that it is installed, and install pip as well:
```bash
python3.10 --version
sudo apt install python3-pip
```

### Clone
Download this repository: 
```
git clone https://github.com/natalieagus/lab_banker
```

You should have the following files:
```cpp
lab_banker/
  |- test_files/
      |- q0_answer.txt
      |- q0.txt
      |- q1_answer.txt
      |- q1.txt
      |- q2_answer.txt
      |- q2.txt
      |- q3_answer.txt
      |- q3.txt
      |- q4_answer.txt
      |- q4.txt  
      |- q5_answer.txt
      |- q5.txt       
  |- .gitignore
  |- banker_test.py
  |- banker.py
  |- README.md   
```

You will be required to modify only certain sections in `banker.py`. 

Leave ALL other files untouched. Also, do not `print` anything else in `banker.py`.
{:.error}

# Banker's Algorithm

## Prerequisite

Suppose we have `N` processes competing for `M` different types of resources. In order for the Banker's Algorithm to work, need to know the <span style="color:#f7007f;"><b>maximum</b></span> demand of process `i` for resource `j` instances. 

We also need to represent the amount of resources available per-type-per-process using the following data structures:
1. available: 1 by M vector 
  * `available[i]`: the available instances of resource `i`
2. max: N by M matrix
  * `max[i][j]`: maximum demand of process `i` for resource `j` instances
3. allocation: N by M matrix
  * `allocation[i][j]`: current allocation of resource `j` instances for process `i`
4. need: N by M matrix
  * `need[i][j]`: how much more of resource `j` instances might be needed by process `i`

For example, the following arrays display the `available`, `max`, `allocation`, and `need` values of a current state of a system with 5 processes: `P0`, `P1`, `P2`, `P3`, and `P4`, and 3 types of resources: `A`, `B`, `C`; 

<img src="/50005/assets/images/lab3/1.png"  class="center_seventy"/>

## Banker.py
Open `Banker.py` and give it a quick read. 

There are a few methods defined in `Class Banker`:
1. `set_maximum_demand`: implemented for you, this populates the corresponding row of `max` matrix
2. `request_resources`: request resource algorithm implementation. <span style="color:#f77729;"><b>Task 1 is here</b></span>.
3. `release_resources`: releases resources borrowed by a customer. Assume release is valid for <span style="color:#f77729;"><b>simplicity</b></span>
4. `check_safe`: safety check algorithm implementation. <span style="color:#f77729;"><b>Task 2 is here</b></span>.
5. `print_state` and `run_file`: functions to load the input files and printing the states of the system: `available, max, allocation, need` values. 

## Input Format
The inputs to `banker.py` are inside `test_files/`. All `.txt` files named as `qi.txt` are various input files. 

The format are as follows:
1. The first three lines contain values of `n` (number of processes), `m` (number of resource type), and the contents of the `available` vector. 
2. For lines starting with `c`, it signifies the `max` demand of a process for ALL resource types.
   * The format is `c,pid,rid_1 rid_2 rid_3 ...` 
3. For lines starting with `r`, it signifies a request by a process. 
   * The format is: `r,pid,rid_1 rid_2 rid_3 ...`
4. For lines starting with `f`, it signifies a release of resources by a process.
   * The format is: `f,pid,rid_1 rid_2 rid_3 ...` 
5. For lines with just `p`, we print the state of the system 

The lines in the input are read from top to bottom, and are treated as incoming requests or releases by each process <span style="color:#f77729;"><b>as time progresses</b></span>. 

For instance, open `test_files/q0.txt`:
```
n,3
m,3
a,0 0 0
c,0,2 2 4
c,1,2 1 3
c,2,3 4 1
r,0,1 2 1
r,1,2 0 1
r,2,2 2 1
p
```
From the first two lines, we know that there are 3 distinct processes (`n`) competing for 3 different types of resources (`m`). 
* Let's name them as P0, P1, and P2 so that it is easier for us to address them. 
* Also label each of the 3 resources as A, B, and C. 

From the third line, we know that NONE of the resources are available: `available = [0,0,0]`. 

In the fourth to sixth line, we can initialise the `max` matrix as follows:
```
max = [ [2,2,4]
        [2,1,3]  
        [3,4,1] ]
```
This means `P0` will need <span style="color:#f77729;"><b>at most</b></span> 2 instances of Resource A, 2 instances of Resource B, and 4 instances of Resource C during its lifetime of execution. Similar logic for P1 and P2. 

In the seventh line `r,0,1 2 1`, P0 starts requesting for 1 instance of Resource A, 2 instances of Resource B, and 1 instance of Resource C simultanously. 

<span style="color:#f77729;"><b>We will need to run the Banker's algorithm now to determine whether this request is `safe`, and to be granted</b></span>.
{:.warning}

This is already done for you inside `run_file` function. Whenever line with `r` is encountered, the function `request_resources` is called. 
* If the request is granted, we need to update `available`, `allocation` and `need`. This is already done for you inside `request_resources` function. 
* If the request is not granted, we simply ignore that request. The values of `available`, `allocation` and `need` remains the same. This is already done for you inside `request_resources` function.

The same is done for all lines starting with `r`. 

At the end, we print the state of the system with `p`. 

In this example, NO resources are available. So obviously we are met with the same state of `available`, `allocation` and `need` after running the file.

You can run the file using the command:
```
python3.10 banker.py test_files/q0.txt
```

The output is as such as expected:
```
Customer 0 requesting
[1, 2, 1]
Customer 1 requesting
[2, 0, 1]
Customer 2 requesting
[2, 2, 1]

Current state:
Available:
[0, 0, 0]

Maximum:
[2, 2, 4]
[2, 1, 3]
[3, 4, 1]

Allocation:
[0, 0, 0]
[0, 0, 0]
[0, 0, 0]

Need:
[2, 2, 4]
[2, 1, 3]
[3, 4, 1]
```

Experiment with the other 5 files as well. Try them one by one and study the result.
```
python3.10 banker.py test_files/q1.txt
python3.10 banker.py test_files/q2.txt
python3.10 banker.py test_files/q3.txt
python3.10 banker.py test_files/q4.txt
python3.10 banker.py test_files/q5.txt
```



