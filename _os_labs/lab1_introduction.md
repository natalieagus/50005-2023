---
title: Introduction to the Command Line Interface
permalink: /os_labs/lab1_intro
key: labs-cli
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
# Overview

An operating system Kernel’s job is to provide **services** so that softwares can communicate with hardware. It manages computer system hardware, memory, and processes (among all others).  Its services are exposed via the system call interface, which in itself is wrapped in C standard Library to provide an API that user applications can interact with. Various user applications are made so that the OS services are usable to humans. These services are those you encounter daily when using a computer, for example: 
* File management (rename, create, list files, delete, etc), 
* Process management (run and terminate), 
* System diagnostics (RAM used, CPU %, disk space), 
* I/O operations like printing, reading from disk, communication via network, resource management (overclock speed, VM size), 
* Protection (security and permission settings), etc. 

# The Shell

A **shell** is one of these user applications that acts as an interface to allow users to **access** OS services. It is named shell because it is seen as an *“outer”* layer around the OS kernel. OS shells are made either in a form of command-line interface (**CLI**, also known as *terminal*) where users can **provide commands** via **text**, or graphical user interface (**GUI**) where users can provide commands via mouse clicks. In this lab, we are going to learn a little bit about the command-line interface, bash scripting, and makefiles. 

You would need to install any POSIX-compliant OS before coming to this lab. The guide is provided in our course handout. 
{:.warning}

In order for us to be able to use CLI, we need to be familiar with their **commands** and their calling **syntax**. In particular, we are concerned with **UNIX-type shells** (POSIX is an IEEE standard that acts as a standard UNIX version) in this course. 
* **Open** your terminal/command line window. 
* The terminal window in front of you contains a `shell`, which enables you to use commands to access OS services.


# Submission
The total marks for this lab is **20**. Please answer the questionnaire provided on eDimension Week 1. You are to score any **20 points** for this lab.  


# Task 1
`TASK 1:`{:.info} To find your current shell, type the command: `ps -p $$`

```java
bash-3.2$ ps -p  $$
  PID TTY           TIME CMD
70846 ttys003    0:00.01 bash
bash-3.2$
```
Bash shell ([bash](https://en.m.wikipedia.org/wiki/Bash_(Unix_shell)) is used in the screenshot above. There are other shells as well: [z-shell](https://en.m.wikipedia.org/wiki/Z_shell)) or [fish](https://en.m.wikipedia.org/wiki/Fish_(Unix_shell)). Which one to choose? It is entirely up to you. 