---
title: TinyOS I/O
permalink: /os_labs/lab2_intro
key: labs-tinyos
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

In this lab, you are tasked to write an I/O interrupt handler (**asynchronous** interrupt) and an I/O supervisor call (**synchronous** interrupt) for a very simple OS called the TinyOS using the Beta assembly language that you have learned from 50002 last term. This is closely related to what we have learned last week regarding [hardware interrupt](https://natalieagus.github.io/50005/os_notes/week1_resource#hardware-interrupt) and [system call](https://natalieagus.github.io/50005/os_notes/week2_syscall) as part of Operating System services.

This lab is also closely related to your 50002 materials. In particular, the lecture notes on [Virtual Machine](https://natalieagus.github.io/50002/notes/virtualmachine), and [Asynchronous handling of I/O Devices](https://natalieagus.github.io/50002/notes/asyncio) are closely related to this lab.

Related sections in 50002 [Virtual Machine](https://natalieagus.github.io/50002/notes/virtualmachine):

- [The OS Kernel](https://natalieagus.github.io/50002/notes/virtualmachine#the-operating-system-kernel): The "TinyOS" in this lab represents a very simple kernel that timeshare the execution of three processes: P0, P1, and P2. It keeps track of the execution of these processes via a process table
- [OS Multiplexing and Context Switching](https://natalieagus.github.io/50002/notes/virtualmachine#os-multiplexing-and-context-switching): Round robin execution of P0, P1, and P2 in the lab
- [Asynchronous Interrupt Hardware](https://natalieagus.github.io/50002/notes/virtualmachine#beta-asynchronous-interrupt-hardware): Input from Keyboard and Mouse will interrupt the current process while safely saving the current process' context for later execution
- [Asynchronous Interrupt Handler](https://natalieagus.github.io/50002/notes/virtualmachine#asynchronous-interrupt-handler): the TinyOS is in charge of handling asynchronous Keyboard and Mouse interrupt from the user
- [Trap or Synchronous Interrupt](https://natalieagus.github.io/50002/notes/virtualmachine#trap): A process can ask for user input or OS Services via an ILLOP, thereby forcing a synchronous interrupt or _trap_ for the TinyOS to handle

Related sections in 50002 [Asynchronous handling of I/O Devices](https://natalieagus.github.io/50002/notes/asyncio):

- [The Supervisor Call](https://natalieagus.github.io/50002/notes/asyncio#the-supervisor-call): More details on _trap_
- [Asynchronous Input Handling](https://natalieagus.github.io/50002/notes/asyncio#asynchronous-input-handling) and [Real time I/O Handling](https://natalieagus.github.io/50002/notes/asyncio#real-time-io-handling): More details on _async IO_ due to Keyboard and Mouse interrupt, as well as timer interrupt
