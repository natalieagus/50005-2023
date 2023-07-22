---
title: Introduction
permalink: /ns_labs/lab3_1-intro
key: ns-labs-lab3_1-intro
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

In NS Module 4, we learnt about the role of the Domain Name System (DNS) in Internet naming and addressing. In this lab exercise, we will go deeper into DNS by using specialised network tools to perform and analyse DNS queries.

At the end of this lab exercise, you should be able to:

- Use `dig` to perform DNS queries (e.g. to look up an IP address)
- <span style="color:#f77729;"><b>Read</b></span> and <span style="color:#f77729;"><b>interpret</b></span> DNS records of different types
- Understand how a DNS query is resolved using <span style="color:#f77729;"><b>hierarchy</b></span> and <span style="color:#f77729;"><b>recursion</b></span>
- Observe and understand the effect of <span style="color:#f77729;"><b>caching</b></span> on DNS lookup times
- Use <span style="color:#f77729;"><b>Wireshark</b></span> to trace and read DNS packets sent to and from a machine

## Part 1: Exploring DNS via dig

The Domain Information Groper (`dig`) is commonly used for performing DNS lookups. Here is an example of how it can be used to find information about the host slashdot.org. The results may differ if you run the same query on your machine.

<img src="/50005/assets/images/nslab3/1.png"  class="center_seventy"/>

When the command `dig slashdot.org` is run, dig performs a DNS lookup and displays information about the request and the response it receives. At the bottom of the printout, we can see that the query was sent to the DNS server running on `192.168.2.100`, and that the query took 101 ms to complete. Most of the information that we are interested in can be found in the `ANSWER SECTION`.

The answer section for this query contains a DNS record:

| Server Name    | Expiry (TTL in seconds) | Class | Type | Data           |
| -------------- | ----------------------- | ----- | ---- | -------------- |
| `slashdot.org` | 293                     | IN    | `A`  | `67.251.97.40` |

- We can see that the result is of <span style="color:#f77729;"><b>type</b></span> `A`, an address record.
- It tells us that the IP address for the domain name slashdot.org is `67.251.97.40`.
- The expiry time field indicates that this record is valid for 293 seconds.
- The value of the class field is usually IN (Internet) for all records.

If you’d like to know who’s the authoritative NS for the queried domain, you can add the trace option: `dig slashdot.org +trace`
<img src="/50005/assets/images/nslab3/2.png"  class="center_seventy"/>

The records of type `NS` indicate the names of the DNS servers storing records for a particular <span style="color:#f77729;"><b>domain</b></span>. Here, we can see that the hosts `ns1.dnsmadeeasy.com.` and etc are responsible for providing <span style="color:#f77729;"><b>authoritative responses</b></span> to names in the `slashdot.org` domain.

We can <span style="color:#f77729;"><b>query</b></span> a specific server for information about a host by using the `@` option. For example, to perform a lookup using the DNS server `ns1.dnsmadeeasy.com.`, we can run the command `dig @ns1.dnsmadeeasy.com. slashdot.org.`:

<img src="/50005/assets/images/nslab3/3.png"  class="center_seventy"/>

There are three flags under the header: `qr`, `aa`, and `rd`:

- This means that the message is a query (`qr`), and dig is requesting a recursive lookup (`rd` stands for ‘recursion desired’) and the server is the authoritative name server (`aa` stands for ‘authoritative answer’).
- Not all servers perform recursive lookups due to the heavier load involved, and so you don’t see any `ra` flags here (`ra` stands for ‘recursion available’).

`dig` only prints the <span style="color:#f77729;"><b>final</b></span> result of a recursive search, but you can mimic the individual steps involved by making a query with the `+norecurs` option enabled. For example, to send a non-recursive query to one of the root servers, we enter the command `dig @a.ROOT-SERVERS.NET www.slashdot.org +norecurs`:

<img src="/50005/assets/images/nslab3/4.png"  class="center_seventy"/>

As you can see, the server <span style="color:#f77729;"><b>does not know the answer</b></span> (there’s `0 ANSWER`) and instead provides information about the servers _most likely_ to be able to provide an authoritative answer for the question. In this case, the best that the root server knows is the identities of the servers for the `org.` top-level domain.

### Task 1

`TASK 1:`{:.info} Using dig, find the IP address for thyme.lcs.mit.edu. What is the IP address?

### Task 2

`TASK 2:`{:.info} The dig answer for the previous question includes a record of type CNAME. What does CNAME mean?

### Task 3

`TASK 3:`{:.info} What is the expiration time for the CNAME record?

### Task 4

`TASK 4:`{:.info} Run the following commands to find out what your computer receives when it looks up ‘ai’ and ‘ai.’ in the mit.edu domain. What are the two resulting IP addresses?

1. `dig +domain=mit.edu ai`
2. `dig +domain=mit.edu ai.`

### Task 5

`TASK 5:`{:.info} Find out why the results for both queries are different.

Look up the manual for `dig` to find out what the `+domain` parameter does. Based on the output of the two commands, what is the difference between the DNS searches being performed for `ai` and `ai.`?
