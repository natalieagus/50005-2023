---
title: The Shell
permalink: /ns_labs/lab1_clis
key: labs-cli
license: false
---

In order for us to be able to use CLI, we need to be familiar with their **commands** and their calling **syntax**. In particular, we are concerned with **UNIX-type shells** (POSIX is an IEEE standard that acts as a standard UNIX version) in this course. 
* **Open** your terminal/command line window. 
* The terminal window in front of you contains a `shell`, which enables you to use commands to access OS services.
  
`TASK 1:`{:.info} To find your current shell, type the command: `ps -p $$`

```java
bash-3.2$ ps -p  $$
  PID TTY           TIME CMD
70846 ttys003    0:00.01 bash
bash-3.2$
```
Bash shell ([zsh](https://en.m.wikipedia.org/wiki/Bash_(Unix_shell)) is used in the screenshot above. There are other shells as well: [z-shell](https://en.m.wikipedia.org/wiki/Z_shell)) or [fish](https://en.m.wikipedia.org/wiki/Fish_(Unix_shell)). Which one to choose? It is entirely up to you. 

# Introduction to Commands
The CLI accepts **commands** (the **first** word, e.g. ps, that you type into the CLI is the command), **entered line by line** and it will be executed **sequentially**. There are two types of commands in general, commands **with** and **without** options or arguments.

## Commands Without Options or Arguments
`TASK 2:`{:.info} Try the following basic commands in sequence `date`, `cal`, `pwd`, `who`, `clear`:
1. E.g: date and press enter. You should see today’s date given to you, for example:

```java
bash-3.2$ date
Thu May  5 14:08:51 +08 2022
bash-3.2$
```

2. Do the same thing with `cal, pwd, who, clear`. 
   * Figure out what each command does using `man`
   * You can use `man <command>` and press `q` to quit anytime. 


## Commands With Options or Arguments
The commands you have typed above are those that **do not** require options or more arguments (although they do accept these too). Some commands require more input arguments.

For example, the command ps shows the list of processes for the current shell, while `ps -x` shows all processes that are owned by the current user even when it doesn’t have a **controlling terminal**. 

```java
bash-3.2$ ps
  PID TTY           TIME CMD
61581 ttys000    0:01.80 /bin/zsh -l
61605 ttys000    0:00.00 /bin/zsh -l
61652 ttys000    0:00.00 /bin/zsh -l
61653 ttys000    0:00.04 /bin/zsh -l

bash-3.2$ ps -x
  PID TTY           TIME CMD
  393 ??         0:02.43 /System/Library/Frameworks/LocalAuthentication.framework/Support/coreauthd
  394 ??         1:29.34 /usr/sbin/cfprefsd agent
  398 ??         0:00.05 /System/Library/Frameworks/ColorSync.framework/Versions/A/XPCServices/com.app
  399 ??        25:02.92 /usr/libexec/UserEventAgent (Aqua)
  401 ??         2:06.64 /usr/sbin/distnoted agent
  403 ??         0:12.27 /System/Library/PrivateFrameworks/CloudServices.framework/Helpers/com.apple.s
  404 ??         2:02.15 /usr/libexec/knowledge-agent
  405 ??         0:06.55 /usr/libexec/lsd
  406 ??         3:05.55 /usr/libexec/trustd --agent
  407 ??        15:44.58 /usr/libexec/secd
  409 ??         0:02.05 /System/Library/CoreServices/sharedfilelistd
  410 ??        40:41.61 /System/Library/PrivateFrameworks/CloudKitDaemon.framework/Support/cloudd
  411 ??         0:00.19 /System/Library/CoreServices/backgroundtaskmanagementagent
  412 ??         0:05.30 /System/Library/PrivateFrameworks/TCC.framework/Resources/tccd
  413 ??        12:36.92 /usr/libexec/nsurlsessiond
```

Some commands accept single hyphen (-) as options, and some other accepts double hyphens (--), or no hyphen at all. It really depends on **convention**, so be sure to **read the manual properly**. 
{:.info}

For example, the command: `git commit -v -a --amend` stands for:
* Use git to **stage** new changes (`commit`)
* Automatically stage **all** files that have been modified/deleted (`-a`)
* Do it in a **verbose** way, with detailed message highlighting all diff (`-v`)
* **Replace** the tip of the current branch by creating a new commit (`--amend`)

## Path and Current Working Directory

In a **desktop** environment, you have windows, menu bars, the desktop, etc to give **context** to what you are doing (graphically). 

Your **current** **directory** provides context for the commands you run.
{:.red}

For instance, if you double-click a Downloads folder on the GUI, then it will show the content of the Downloads folder for you. Double clicking different folders will give you different results as their **contexts** are different.

In the command line, however, the **context is solely the file system** (week 6 material). When you first open a terminal window and type pwd, you will typically be in your `home` (symbolised as tilde ~) **directory** already (unless you use a different setup for your terminal). 

The file system has many directories, starting from the **ROOT** (symbolised as a single forward slash /) and you can have directories within a directory thus forming a **hierarchy** of directories. Each “level” is separated by the forward slash symbol.
{:.info}

For instance,

`/Users/natalie_agus/Downloads` simply look like this in the window:

<img src="/50005/assets/images/lab1/2.png"  class="center_full"/>

If your directory has spaces in its name, you need to use the **backslash**) to indicate that it is part of the string, eg:
`/Users/natalie_agus/Google\ Drive`

`TASK 3:`{:.info} Find your starting context by running the command `pwd`, followed by `ls`:
* The command `ls` lists **all** files that exist in this **context**, which is the Desktop directory in the example above (`/Users/natalie_agus/Desktop`)
* When you run the command **ls** by itself, it uses your **current** directory as the context, and lists the files that are in the directory you are in.
* You can use the `ls` command to list the files in a directory that's **not** your current directory, e.g: `ls /Users/natalie_agus/Downloads`

Now try another command called `cd` to **change** the current working directory:
* Change the current working directory into any directory that’s accessible from the current context
* The command cd is analogous to **double clicking** a folder in the GUI 
* You can go “back” to one previous directory level using the command `cd ..` (or you can chain it to go two levels up for instance, `cd ../..`

<img src="/50005/assets/images/lab1/3.png"  class="center_full"/>
