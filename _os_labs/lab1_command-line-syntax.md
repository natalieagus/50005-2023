---
title: Introduction to CLI
permalink: /os_labs/lab1_clis
key: labs-cli
license: false
---


The CLI accepts **commands** (the **first** word, e.g. ps, that you type into the CLI is the command), **entered line by line** and it will be executed **sequentially**. There are two types of commands in general, commands **with** and **without** options or arguments.

# Basic Commands
## Commands Without Options or Arguments
### Task 2
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

# Path 
## Basics of Directory
The file system has many directories, starting from the **ROOT** (symbolised as a single forward slash /) and you can have directories within a directory thus forming a **hierarchy** of directories. Each “level” is separated by the forward slash symbol.
{:.info}

For instance,

`/Users/natalie_agus/Downloads` simply look like this in the window:

<img src="/50005/assets/images/lab1/2.png"  class="center_full"/>

If your directory has spaces in its name, you need to use the **backslash**) to indicate that it is part of the string, eg:
`/Users/natalie_agus/Google\ Drive`

### Task 3
`TASK 3:`{:.info} Find your starting context by running the command `pwd`, followed by `ls`:
* The command `ls` lists **all** files that exist in this **context**, which is the Desktop directory in the example above (`/Users/natalie_agus/Desktop`)
* When you run the command **ls** by itself, it uses your **current** directory as the context, and lists the files that are in the directory you are in.
* You can use the `ls` command to list the files in a directory that's **not** your current directory, e.g: `ls /Users/natalie_agus/Downloads`

Now try another command called `cd` to **change** the current working directory:
* Change the current working directory into any directory that’s accessible from the current context
* The command cd is analogous to **double clicking** a folder in the GUI 
* You can go “back” to one previous directory level using the command `cd ..` (or you can chain it to go two levels up for instance, `cd ../..`

<img src="/50005/assets/images/lab1/3.png"  class="center_full"/>

## Environment Variables
Another way of providing context is through something called **environment** **variables**. Tryout the command: `cd $HOME`
* The `$HOME` part is a reference to the HOME variable, and is replaced by the path to your home directory when the command is run. 
* In other words, running cd `$HOME` is the same as running `cd <actual path to your home>`, or `cd ~` (~ is a default **shell variable** that points to the current User’s home directory, and it also has [other usages](https://www.baeldung.com/linux/tilde-bash) that you can read if you’re free).
* To checkout what the value of your `$HOME` variable is, type the command: echo `$HOME`

### Task 4
`TASK 4:`{:.info} To find out your current environment variables, do the following,
1. Enter the command `env`
2. Find the value for `HOME` and `PATH`
3. Now type `echo $HOME` and `echo $PATH`
4. Are the answers from part (2) and (3) the same?

### Task 5
`TASK 5:`{:.info} You can make your own environment variables using the command `export` 
1. Run the command: `export MESSAGE1="This is message 1"`
2. You can now execute `echo $MESSAGE1` and observe get the string output

In the example below, the environment variable `$MESSAGE1` initially did not exist. After we `export` it, we can now print the environment variable `$MESSAGE1`. 

<img src="/50005/assets/images/lab1/4.png"  class="center_seventy"/>
