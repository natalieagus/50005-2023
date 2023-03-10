---
title: Time-of-Check/Time-of-Use Bug (TOCTOU)
permalink: /os_labs/lab3_intro
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

In this lab, you are tasked to investigate a program with TOCTOU (Time of Check - Time of Use) race-condition vulnerability.

The lab is written <span style="color:#f77729;"><b>entirely in C</b></span>, and it is more of an <span style="color:#f77729;"><b>investigative</b></span> lab with fewer coding components as opposed to our previous lab. At the end of this lab, you should be able to:

- Understand what is a <span style="color:#f77729;"><b>TOUTOU bug</b></span> and why is it prone to attacks
- Detect <span style="color:#f77729;"><b>race-condition</b></span> caused by the TOCTOU bug
- Provide a <span style="color:#f77729;"><b>fix</b></span> to this TOCTOU vulnerability
- <span style="color:#f77729;"><b>Examine</b></span> file permissions and modify them
- Understand the concept of <span style="color:#f77729;"><b>privileged programs</b></span>: user level vs root level
- <span style="color:#f77729;"><b>Compile</b></span> programs and make documents with different privilege level
- Understand how <span style="color:#f77729;"><b>sudo</b></span> works
- Understand the difference between <span style="color:#f77729;"><b>symbolic</b></span> and <span style="color:#f77729;"><b>hard</b></span> links

# Background

## Race Condition

A <span style="color:#f77729;"><b>race condition</b></span> occurs when two or more threads (or processes) access and perform <span style="color:#f77729;"><b>non-atomic operations</b></span> on a shared variable (be it across threads or processes) value at the same time. Since we cannot control the <span style="color:#f77729;"><b>order</b></span> of execution, we can say that the threads / processes race to modify the value of the shared variable.

The <span style="color:#f77729;"><b>final</b></span> value of the shared variable therefore can be <span style="color:#f7007f;"><b>non deterministic</b></span>, depending on the particular <span style="color:#f77729;"><b>order</b></span> in which the access takes place. In other words, the cause of race condition is due to the fact that the function performed on the shared variable is non-atomic.

In this lab we are going to <span style="color:#f77729;"><b>exploit</b></span> a program that is <span style="color:#f7007f;"><b>vulnerable to race-condition</b></span>.

- The program alone is single-threaded, so in the absence of an attacker, there’s nothing wrong with the program.
- The program however is vulnerable because an attacker can exploit the fact that the program can be subjected to race-condition.

You will learn about race condition properly in future lectures. This lab is just a preview.
{:.warning}

# Setup

## Installation

Download the files for this lab using the command:

```git
git clone https://github.com/natalieagus/lab_toctou
```

You should find that the following files are given to you:
<img src="/50005/assets/images/lab2/1.png"  class="center_fourty"/>

Now go to the `User/` directory and call `make`. You should find these files in the end.
<img src="/50005/assets/images/lab2/2.png"  class="center_seventy"/>

Do <span style="color:#f7007f;"><b>NOT</b></span> used a shared drive with your Host machine, or a shared external drive with other OS. You will NOT be able to create root files in this case. Clone the above file in /home/<user> directory instead.
{:.error}

## Login as Root User

### Task 1

`TASK 1:`{:.info} To switch user and login as root, you must first set the <span style="color:#f77729;"><b>password</b></span> for the root account:

```console
sudo passwd root
```

You will be prompted for your <span style="color:#f77729;"><b>current user</b></span> password, then set a new password for `root`.

Remember this `root` <span style="color:#f77729;"><b>password</b></span>! You need this later to switch to `root` user. If you have a brainblock, use a simple phrase like: `rotiprata`.
{:.error}

Follow the instructions, and then switch to the root user using the command and the password you just created above for `root`:

```console
su root
```

You will see the following <span style="color:#f7007f;"><b>new prompt</b></span>, indicating now you're logged in as `root`:
<img src="/50005/assets/images/lab2/3.png"  class="center_seventy"/>

Root user is the user with the highest (administrative) privilege. It has <span style="color:#f7007f;"><b>nothing to do with Kernel Mode</b></span>. Processes spawned while logged in as Root still runs on <span style="color:#f77729;"><b>User Mode</b></span>.
{:.warning}

### Task 2

`TASK 2:`{:.info} Create files while logged in as `root`.

Now while logged in as `root`, navigate to `/FilesForRoot`, and type `make`. You should see a new directory called `Root/` created with the following contents:

<img src="/50005/assets/images/lab2/4.png"  class="center_seventy"/>

It is <span style="color:#f7007f;"><b>important</b></span> to check that the newly created files belong to the user root as shown in yellow above (beside the date is the `owner` of the file).

## adduser

### Task 3

`TASK 3:`{:.info} Create 2-3 other users.

In order to proceed with this lab, you need to <span style="color:#f77729;"><b>login</b></span> as `root` and create a few <span style="color:#f77729;"><b>new</b></span> users before proceeding.

For the first user, do the following (the username must be `test-user-0`):

```console
adduser test-user-0
```

Give it any password you like (preferably a good one, like `LDcwzD&#6JKr`), and then add it to the `sudo` group:

```console
adduser test-user-0 sudo
```

<img src="/50005/assets/images/lab2/14.png"  class="center_seventy"/>

Then add a few more with different names: e.g `test-user-0`, `test-user-1`, with any appropriate password of your choice.

## su

### Task 4

`TASK 4:`{:.info} Switch account as any of these new users to ensure that you've created new users successfully.

You can switch to your new user by using the command `su <username>`:

<img src="/50005/assets/images/lab2/15.png"  class="center_seventy"/>

Once you're done, <span style="color:#f77729;"><b>switch back</b></span> to `root` again using your `root` password you've set above.

### Task 5

`TASK 5:`{:.info} Check `/etc/shadow`.

Once you're done, you should enter the command `cat /etc/shadow` and see that your newly created users are at the bottom of the file, with some hash values (actual values are different depending on the password you set for these users).

<img src="/50005/assets/images/lab2/16.png"  class="center_seventy"/>

### Task 6

`TASK 6:`{:.info} Return to your original user account.

You can switch back to your original normal user account by using the same `su <username>` command:
<img src="/50005/assets/images/lab2/5.png"  class="center_seventy"/>

Now go up one level by typing `cd ..` and attempt to <span style="color:#f7007f;"><b>delete</b></span> `Root/` directory while logged in as your original <span style="color:#f77729;"><b>username</b></span>. You will find such permission denied message:

<img src="/50005/assets/images/lab2/6.png"  class="center_seventy"/>

What happened?
{:.warning}

## File Permission

The reason you faced the <span style="color:#f7007f;"><b>permission denied</b></span> error is because your username doesn't have the correct <span style="color:#f7007f;"><b>permission</b></span> to edit the `Root/` directory.

<img src="/50005/assets/images/lab2/7.png"  class="center_fifty"/>

Notice that a <span style="color:#f77729;"><b>directory</b></span> is an <span style="color:#f77729;"><b>executable</b></span>, indicated by the ‘x’ symbol.

## The SUID bit

SUID stand for Set Owner User ID. A file with <span style="color:#f77729;"><b>SUID</b></span> set always <span style="color:#f77729;"><b>executes</b></span> as the user who <span style="color:#f7007f;"><b>owns</b></span> the file, regardless of the user passing the command.
{:.warning}

List the file permission for all files inside `Root/` directory:
<img src="/50005/assets/images/lab2/8.png"  class="center_seventy"/>

Notice that instead of an `x`, we have an `s` type of permission listed on the two progs: `vulnerable_root_prog` and `rootdo`.

**This is called the SUID bit.**
{:.error}

The <span style="color:#f7007f;"><b>SUID</b></span> bit allows <span style="color:#f77729;"><b>normal</b></span> user to gain <span style="color:#f77729;"><b>elevated privilege</b></span> (again this does <span style="color:#f7007f;"><b>NOT</b></span> mean Kernel Mode, just privilege levels among regular users) when executing this program.

- If a normal user executes this program, this program runs in root <span style="color:#f77729;"><b>privileges</b></span> (basically, the creator of the program)

### Task 7

`TASK 7:`{:.info} Reading protected file using regular user account.

While logged in as your original user account, try to read the file `/etc/shadow`:

```console
cat /etc/shadow
```

You will be met with <span style="color:#f77729;"><b>permission denied</b></span> because this file can only be read by `root` user, and other users in the <span style="color:#f77729;"><b>same group</b></span>, as shown in the file details below;
<img src="/50005/assets/images/lab2/10.png"  class="center_seventy"/>

What group does `root` belong to? What about the user account in question (ubuntu in example above)? You can find out using the command `groups`:

<img src="/50005/assets/images/lab2/22.png"  class="center_seventy"/>

### Task 8

`TASK 8:`{:.info} Gain privilege elevation.

Now, run the following command. We assume that your <span style="color:#f77729;"><b>current working directory</b></span> is at `/lab_toctou` directory. If not, please adjust accordingly.

```console
./Root/rootdo cat /etc/shadow
```

When prompted, type the word `password`, and then press enter.

You will find that you can now read this file:
<img src="/50005/assets/images/lab2/9.png"  class="center_seventy"/>

The <span style="color:#f77729;"><b>reason</b></span> you can now successfully read the file `/etc/shadow` is because `rootdo` <span style="color:#f77729;"><b>has the SUID bit</b></span>. Any other program that is <span style="color:#f77729;"><b>executed</b></span> by `rootdo` will run with `root` (`rootdo` creator) privileges and <span style="color:#f77729;"><b>not</b></span> the regular user.

You can open `rootdo.c` inside `/lab_toctou/FilesForRoot/` to examine how it works, especially this part where it just simply checks that you have keyed in `password` and proceed to `execvp` (execute) the input command:

```java
    if (!strcmp(password, "password"))
    { // on success, returns 0
        printf("Login granted\n");
        int pid = fork();
        if (pid == 0)
        {
            printf("Fork success\n");
            wait(NULL);
            printf("Children returned\n");
        }
        else
        {
            if (execvp(execName, argv_new) == -1)
            {
                perror("Executable not found\n");
            }
        }
    }
```

In short, as the <span style="color:#f7007f;"><b>SUID</b></span> bit of rootdo program is set, it <span style="color:#f7007f;"><b>always runs with root privileges</b></span> <span style="color:#f77729;"><b>regardless</b></span> of which user executes the program.

While rootdo seems like a <span style="color:#f77729;"><b>dangerous</b></span> program, don’t forget that the <span style="color:#f77729;"><b>root</b></span> itself was the one who made it and set the SUID bit in the first place, so yes it is indeed<span style="color:#f77729;"><b> meant to run that way</b></span>.

## sudo

This is in fact how your <span style="color:#f7007f;"><b>sudo</b></span> program works. When you typo `sudo <command>`, it prompts you for your <span style="color:#f77729;"><b>password</b></span>, then the program checks whether the user is verified, before executing with root privileges. In our little `rootdo` example, we just use hardcoded `password` to proceed.
{:.warning}

Setting of `rootdo`'s <span style="color:#f77729;"><b>SUID</b></span> bit is done in the `makefile` inside `FilesForRoot` that you execute earlier while logged in as root,

```makefile
# refresh the root files and root log file
# Set UID for rootprog and rootdo
setup:
	chmod u+s ../Root/vulnerable_root_prog
	chmod u+s ../Root/rootdo
```

The command `chmod u+s filename` sets the `SUID` bit of that `filename`.
