---
title: DNS Hierarchy 
permalink: /ns_labs/lab3_2-hierarchy-caching
key: ns-labs-lab3_2-hierarchy-caching
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


## norecurse
In the previous section, you ran `dig` without changing the default options. This causes `dig` to perform a <span style="color:#f77729;"><b>recursive</b></span> lookup if the DNS server being queried supports it. In this part, you will *trace* the intermediate steps involved in a performing recursive query by beginning at a `root` server and <span style="color:#f77729;"><b>manually</b></span> going through the DNS hierarchy to resolve a host name. You can obtain a list of all the root servers by running the command `dig . NS`

### Task 6 
`TASK 6:`{:.info} Use `dig` to query the `c` DNS root server for the IP address of `lirone.csail.mit.edu` <span style="color:#f77729;"><b>without</b></span> using recursion. What is the command that you use to do this?

### Task 7 
`TASK 7:`{:.info} Go through the DNS hierarchy from the root until you have found the IP address of `lirone.csail.mit.edu`.

You should disable recursion and follow the referrals manually. Take note of all the commands that you use, and addresses that you found. 

## DNS Caching
### Task 8 
`TASK 8:`{:.info} <span style="color:#f77729;"><b>Without</b></span> using recursion, query your default (local) DNS server for information about `www.dmoz.org`. Answer the following questions for your own practice:
* What is the command that you used? 
* Did your default server have the answer in its cache?<span style="color:#f77729;"><b> </b></span>How did you know?
* How <span style="color:#f77729;"><b>long</b></span> did the query take?

### Task 9 
`TASK 9:`{:.info} Query your default DNS server for information about the host in the previous question, using the recursion option this time. How long did the query take? (is it faster or slower than the previous task?)

### Task 10 
`TASK 10:`{:.info} Query your default DNS server for information about the same host without using recursion. How long did the query take? 
> Has the cache served its purpose? Think about the reason(s) why. 

### Task 11 
`TASK 11:`{:.info} Try to query about `www.singtel.com` as such: `dig www.singtel.com +nocmd +noall +answer`. Take note of the output. Then, <span style="color:#f77729;"><b>wait</b></span> a few seconds and type the query <span style="color:#f77729;"><b>again</b></span>. 
> Repeat the query a few times after waiting for a few seconds. Also, search what each <span style="color:#f77729;"><b>query option</b></span> does. 

You might have such output:
<img src="/50005/assets/images/nslab3/7.png"  class="center_full"/>

Do you observe a *pattern* in the `TTL` field? (e.g: `TTL` value is reducing or increasing, or wrapped around).

