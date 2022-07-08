---
title: Ping and Traceroute
permalink: /ns_labs/lab1_1
key: ns-labs-lab1_1
layout: article
nav_key: ns_labs
sidebar:
   nav: ns_labs
license: false
aside:
   toc: true
show_edit_on_github: false
show_date: false
---

In this lab exercise, you will learn how to use `ping` and `traceroute` to measure round trip times and find network routes.

<span style="color:#f77729;"><b>Learning Objectives</b></span>:
* <span style="color:#f77729;"><b>Understand</b></span> how the ping and traceroute utilities work.
* Use the ping utility to <span style="color:#f77729;"><b>measure</b></span> network round trip times.
* Use the traceroute utility to find network <span style="color:#f77729;"><b>routes</b></span>.
* Observe and understand the effects of <span style="color:#f77729;"><b>varying</b></span> packet sizes on delays experienced.

You will `need` `ping` and traceroute to be installed on your OS. Most Ubuntu or macOS installations should already include `ping` by default. You can install traceroute by running `sudo apt install traceroute` from the command line.

# RTT Measurement using ping

The `ping` utility is one of the most widely-used network utilities. It enables you to <span style="color:#f77729;"><b>measure</b></span> the time that it takes for a packet to travel through the Internet to a remote host *and back*.

The `ping` utility works by sending a **short** message, known as an <span style="color:#f77729;"><b>echo request</b></span> to a remote host using the Internet Control Message Protocol (ICMP). 
{:.warning}

> When a host that supports ICMP receives an echo-request message, it replies by sending an echo-response message back to the originating host.

In this first part of this lab exercise, you will use the `ping` utility to send echo requests to a number of *different* hosts. In many of the exercises, you will be referring to hosts using their <span style="color:#f77729;"><b>domain name</b></span> rather than their IP addresses[^1]. For more information about ping, you can look up its manual page by running `man ping` from the command line.

The following info is relevant for the next few tasks:
* `ping netflix.com` is the easiest and simplest way to ping a server. It will continuously send packets and print out the response (if any). You may press `ctrl+c` to terminate it. 
* It supports several options:
  * `-c [no_of_packets]`: specify the number of packets that should be sent by `ping` before terminating (otherwise it will continue forever until `SIGINT`[^2] is sent by `ctrl+c`).
  * `-s [packet_size]`: set packet size. The default is 56.
  * `-i [seconds_interval]`: interval of ping packets sent to the destination

## Round-Trip Time
The `ping` utility can be used to measure the round-trip time (RTT).

Round-trip time (<span style="color:#f77729;"><b>RTT</b></span>) is the duration, measured in milliseconds, <span style="color:#f77729;"><b>from</b></span> when a browser sends a request <span style="color:#f77729;"><b>to</b></span> when it receives a response from a server. 
{:.warning}

RTT is one of the <span style="color:#f77729;"><b>key performance metric</b></span> for web applications.

### Task 1 
`TASK 1:`{:.info} Use `ping` to send 10 packets (56 bytes each) to each of the following hosts, and there should be an interval of 5 seconds between each packet sent:
* www.csail.mit.edu
* www.berkeley.edu
* www.usyd.edu.au
* www.kyoto-u.ac.jp

> **Note:** The size of each packet is 56 bytes by default, but you may observe that the actual size of the packet is <span style="color:#f77729;"><b>larger</b></span> than 56 bytes. You can look up the manual for ping to understand <span style="color:#f77729;"><b>why</b></span> such a discrepancy exists.

Fill up the table below, and head to edimension to key in your answer.

Website | Successfull %| Min RTT | Ave RTT | Max RTT
---------|----------|---------|----------|---------
 www.csail.mit.edu |  |  |  | 
 www.berkeley.edu |  |  |  | 
 www.usyd.edu.au |  |  |  | 
 www.kyoto-u.ac.jp |  |  |  | 
 
Also, go to this [online ping test site](https://tools.keycdn.com/ping) and ping www.csail.mit.edu
> From whom do you receive replies? You can get the IP address and use the command `whois [ip_address]`

### Task 2 
`TASK 2:`{:.info} Repeat the exercise from Task 1 using packet sizes of 512 and 1024 bytes. Record the <span style="color:#f77729;"><b>minimum</b></span>, <span style="color:#f77729;"><b>average</b></span>, and <span style="color:#f77729;"><b>maximum</b></span> round trip times for each of the packet sizes with a table like the previous task, and head to edimension to key in your answer.

> <span style="color:#f77729;"><b>Why</b></span> are the minimum round-trip times to the same hosts *different* when using 56, 512, and 1024–byte packets?

## Unanswered pings
### Task 3 
`TASK 3:`{:.info} Use ping to send 100 packets to the following host: www.wits.ac.za 

Each packet should have a size of 56 bytes, and there should be an interval of 5 seconds between each packet sent.

<span style="color:#f77729;"><b>Record</b></span> the percentage of the packets sent that resulted in a successful response for each host. 

> What are some possible reasons why you may not have received a response? (Be sure to check the host in a <span style="color:#f77729;"><b>web browser</b></span>).








[^1]:
      A domain name is an easy-to-remember alias used to access websites. For example, we access netflix by typing netflix.com and not the actual netflix server's public IP address. For more information, see [here](https://www.cloudflare.com/en-gb/learning/dns/glossary/what-is-a-domain-name/).

[^2]:
      In POSIX-compliant OS, the default action for `SIGINT`, `SIGTERM`, `SIGQUIT`, and `SIGKILL` is to terminate the process. However, `SIGTERM`, `SIGQUIT`, and `SIGKILL` are defined as signals to terminate the process, but `SIGINT` is defined as an <span style="color:#f77729;"><b>interruption</b></span> requested by the <span style="color:#f77729;"><b>user</b></span>.
