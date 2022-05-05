---
title: Path and Directory
permalink: /ns_labs/lab1_path
key: labs-cli
license: false
---


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
