---
title: Introduction to the Command Line Interface
permalink: /ns_labs/lab1_introduction
key: labs-cli
license: false
---
# Overview

An operating system Kernel’s job is to provide **services** so that softwares can communicate with hardware. It manages computer system hardware, memory, and processes (among all others).  Its services are exposed via the system call interface, which in itself is wrapped in C standard Library to provide an API that user applications can interact with. Various user applications are made so that the OS services are usable to humans. These services are those you encounter daily when using a computer, for example: 
* File management (rename, create, list files, delete, etc), 
* Process management (run and terminate), 
* System diagnostics (RAM used, CPU %, disk space), 
* I/O operations like printing, reading from disk, communication via network, resource management (overclock speed, VM size), 
* Protection (security and permission settings), etc. 


A **shell** is one of these user applications that acts as an interface to allow users to **access** OS services. It is named shell because it is seen as an *“outer”* layer around the OS kernel. OS shells are made either in a form of command-line interface (**CLI**, also known as *terminal*) where users can **provide commands** via **text**, or graphical user interface (**GUI**) where users can provide commands via mouse clicks. In this lab, we are going to learn a little bit about the command-line interface, bash scripting, and makefiles. 

You would need to install any POSIX-compliant OS before coming to this lab. The guide is provided in our course handout. 
{:.warning}

The total marks for this lab is **15**.

# Submission

[TBC]