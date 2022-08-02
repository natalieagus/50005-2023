---
title: Wireshark
permalink: /ns_labs/lab3_3-wireshark
key: ns-labs-lab3_3-wireshark
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

<span style="color:#f77729;"><b>Wireshark</b></span> is a powerful tool used to capture packets sent over a network and analyse the content of the packets retrieved. The file [dnsrealtrace.pcapng](https://drive.google.com/file/d/118Z03KnN7mNchsIs3G-DUdtf1zJV3NVI/view?usp=sharing) contains a <span style="color:#f77729;"><b>trace</b></span> of the packets sent and received when a web page is downloaded from a web server over the SUTD network. 

> In the process of downloading the web page, DNS is used to find the IP address of the server.


## Observe Wireshark Capture File
Open the dnsrealtrace.pcapng in Wireshark and answer the following questions. You can refer to Wireshark tutorial [here](https://drive.google.com/file/d/12zi50lKYTf6ebXQNbUJsstc_BBSWO6X6/view?usp=sharing) before proceeding, but most things are self-explanatory. 

After opening the file, you should have this interface:

<img src="/50005/assets/images/nslab3/5.png"  class="center_full"/>


### Task 12 
`TASK 12:`{:.info} <span style="color:#f77729;"><b>Locate</b></span> the DNS query and response messages. Are they sent over `UDP` or `TCP`?  
> Which numbers are these DNS query packets? Hint: look under protocol <span style="color:#f77729;"><b>DNS</b></span>

### Task 13 
`TASK 13:`{:.info} What is the <span style="color:#f77729;"><b>destination</b></span> port for the DNS *query* message? What is the <span style="color:#f77729;"><b>source</b></span> port of the DNS *response* message?

### Task 14 
`TASK 14:`{:.info} What is the IP address to which the DNS query message was sent? Run `scutil --dns` to determine the `IPv4` address of your *local* DNS server. Are these two addresses the same? 

### Task 15 
`TASK 15:`{:.info} Examine the <span style="color:#f77729;"><b>second</b></span> DNS query message in the Wireshark capture. What <span style="color:#f77729;"><b>type</b></span> of DNS query is it? 
* Does the query message contain any answers?

Then examine the second DNS **response** message. 
* How many answers are provided? 
* What does each of these answers contain?

### Task 16 
`TASK 16:`{:.info} Locate a `TCP SYN` packet sent by your host subsequent to the above (second) DNS response. 

This packet opens a `TCP` <span style="color:#f77729;"><b>connection</b></span> between your host and the web server. Does the <span style="color:#f7007f;"><b>destination</b></span> IP address of the `SYN` packet correspond to any of the IP addresses provided in the DNS response message?

## Optional Activity

Capturing packets for packet analysis with wireshark:
1. Once the program is launched, select the <span style="color:#f77729;"><b>network interface</b></span> to capture and click on the *sharkfin* icon at the top left of the application right under the menu bar to begin capturing packets. If you click on each packet, you can see each layer's header and the application layer payload.
<img src="/50005/assets/images/nslab3/6.png"  class="center_full"/>

2. To explore the interface, mention the interface (e.g. `eth0`, `wlan`) in the capture option.

3. There are display filters to analyse the packets.
   * <span style="color:#f77729;"><b>Protocols</b></span>: TCP, UDP, ARP, SMTP, etc.
   * Protocol <span style="color:#f77729;"><b>fields</b></span>: port, src.addr, length, etc. (E.g. `ip.src == 192.168.1.1`)
   * For more detailed instructions on Wireshark, refer to its [official homepage](https://www.wireshark.org/)
