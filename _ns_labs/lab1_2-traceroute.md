---
title: Traceroute
permalink: /ns_labs/lab1_2
key: ns-labs-lab1_2
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

The `traceroute` utility is another useful network utility. It enables you to trace the route taken by a packet from your machine to a remote host. 

Note that if traceroute doesn’t work on your VM, you may:
* Add the `-I` option: `traceroute -I [ip]`
* Or, use results from `tracert` (assuming Windows is your host OS). 


Here is an example of the output produced when traceroute is used to trace the route taken by a packet to www.mit.edu:

<img src="/50005/assets/images/nslab1/1.png"  class="center_seventy"/>


Each line in the output begins with a <span style="color:#f77729;"><b>host</b></span> on the route from your computer to www.mit.edu, followed by the round-trip time (<span style="color:#f77729;"><b>RTT</b></span>) for <span style="color:#f7007f;"><b>3</b></span> packets sent to that host. 

For more information about `traceroute`, you can look up its manual page by running `man traceroute` from the command line.

### Task 4 
`TASK 4:`{:.info} Find out how `traceroute` works. You will need this to answer several questions on eDimension. 

## Route Asymmetries
The route taken to send a packet from your  machine to the remote host machine is <span style="color:#f77729;"><b>not always the same</b></span> with the route taken to send a packet from the remote machine *back* to you.

In this exercise, you will run traceroute in two <span style="color:#f77729;"><b>opposite</b></span> directions. First, you will run `traceroute` on a remote host to see the route taken <span style="color:#f77729;"><b>to your network</b></span>. Then, you will also run `traceroute` from your computer to see the route taken to that host.

### Task 5 
`TASK 5:`{:.info} Find out your computer’s public IP address. (Hint: You can use a website like [this](http://www.whatismypublicip.com/), or search for “what is my ip” using Google’s search engine.)

### Task 6 
`TASK 6:`{:.info} Visit [this link](https://www.uptrends.com/tools/traceroute) in your web browser. 
* Enter your computer’s public IP address, 
* Select the “from Location”, and follow the steps shown in site for at least <span style="color:#f7007f;"><b>three</b></span> locations namely: New York, Amsterdam, Tokyo.
* Then, click “Start Test” to start a traceroute to your computer. 
* Take a <span style="color:#f77729;"><b>screenshot</b></span> of the output

> If the output shows that the packet *does not reach your IP* (request timed out), think about a reason or two on why this is so.


### Task 7 
`TASK 7:`{:.info} After `traceroute` finishes running, you should be able to view the route taken from <span style="color:#f77729;"><b>specified locations</b></span> to your network. 
* Record the IP address of the <span style="color:#f77729;"><b>first</b></span> hop (hop 1), which will be used in the next step. 

In the screenshot below, that will be 31.204.145.131 for example.

<img src="/50005/assets/images/nslab1/2.png"  class="center_seventy"/>

You’re free to use other similar sites if [the site suggested above](https://www.uptrends.com/tools/traceroute) is blocked in your network, or if you have two devices with different IPs (e.g: one uses VPN), then you can also traceroute each other’s IP addresses. 
{:.warning}

You can <span style="color:#f77729;"><b>check</b></span> who  that remote host is using the command `whois [ip address]`, for instance,  31.204.145.131 is indeed described as being in Tokyo.

<img src="/50005/assets/images/nslab1/4.png"  class="center_seventy"/>

### Task 8 
`TASK 8:`{:.info} On your computer, run `traceroute` using the IP address recorded in the previous step as the remote destination.

For instance,
<img src="/50005/assets/images/nslab1/3.png"  class="center_seventy"/>


## Final Thoughts

Is there anything *unusual* in the output of **Task 8**? Are the <span style="color:#f77729;"><b>same</b></span> routers traversed in both directions? 
> If no, why so?




