---
title: CSEShell
permalink: /assignments/pa1_1_intro
key: assignments-pa1_1_intro
layout: article
nav_key: assignments
sidebar:
  nav: assignments
license: false
aside:
  toc: true
show_edit_on_github: false
show_date: false
---

In this assignment, you are tasked to create a <span style="color:#f77729;"><b>shell</b></span> as well as a <span style="color:#f77729;"><b>daemon</b></span> process, both of which are the common applications of `fork()`.

The assignment is written entirely in C. At the end of this assignment, you should be able to:

- <span style="color:#f77729;"><b>Create</b></span> a shell and wait for user input
- Write several other <span style="color:#f77729;"><b>system programs</b></span> that can be invoked by the shell
- <span style="color:#f77729;"><b>Parse</b></span> user input and invoke `fork()` with the appropriate program
- Create a program that results in a <span style="color:#f77729;"><b>daemon</b></span> process
- Use your shell to keep track of the state of your daemon processes

You may complete this assignment in <span style="color:#f7007f;"><b>pairs</b></span>. Indicate your partner's name in the google sheet provided in our course handout.
{:.info}

## Starter Code

You might want to run this assignment in your **POSIX compliant OS**, using `gcc` version 10.X or below. **Using version 11.X and above** might be _okay_, but the bot is running `gcc 10` version so if you use some newer fancy stuffs then the bot might not be able to run it. You can try first.
{:.error}

You should have joined the GitHub Classroom and obtain the starter code for this assignment there. The link can be found in the Course Calendar portion of your Course Handout.

<!-- Download the starter code:
`git clone https://github.com/natalieagus/pa1.git`

This will result in a directory called `pa1`.

### Create a Github Remote Repo
Open your web browser in your host OS and create a <span style="color:#f7007f;"><b>PRIVATE repository</b></span> called `pa_1`. Make sure you create the `master` branch and push to this <span style="color:#f7007f;"><b>master</b></span> branch for this assignment (and not use `main`!). You also need to [make master the default branch](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-branches-in-your-repository/changing-the-default-branch).

Then, cd to `/pa1/` that you have downloaded, and <span style="color:#f7007f;"><b>add</b></span> this project to your own <span style="color:#f7007f;"><b>private</b></span> repo:

```bash
git remote remove origin
git remote add origin https://github.com/[your_github_username]/pa_1.git
git push -u origin master
```

<img src="/50005-2023/assets/images/pa1/16.png"  class="center_seventy"/>

If you only have `main` branch, the third command will fail. You need to create the master branch first and switch there:
```
git branch master
git checkout master
git push -u origin master
```

From now on, just stay at `master`. You're free to create other branches but `master` is the branch which we will grade.

You will be required to make a `commit`{:.error} after each Task in this PA1. This is part of our <span style="color:#f7007f;"><b>grading</b></span> requirement. We want to see that you actually make good practices and perform periodic commits.

### Add `natalieagus-sutd` as collaborator
In your pa1 repo, <span style="color:#f77729;"><b>invite</b></span> `natalieagus-sutd` as your collaborator. This is so that your repo can remain private and and we can pull your submissions for grading when it is due.   -->

### PA1 Files

You should have the following files:

```cpp
pa1/
  |- bin/source/
      |- check_daemon.c
      |- count_line.c
      |- display.c
      |- find.c
      |- listdir_all.c
      |- listdir.c
      |- shell.c
      |- shell.h
      |- summond.c
      |- system_program.h
  |- files/
      |-combined.txt
      |- file1.txt
      |- file2.txt
      |- intermediate.txt
      |- lorem_ipsum.txt
      |- notes.pdf
      |- oneline.txt
      |- paragraph.txt
      |- ss.png
  |- .gitignore
  |- README.md
  |- makefile
```

You will only need to modify four files for this assignment: `shell.c`, `countline.c`, `summond.c`, and `checkdaemon.c`.

<span style="color:#f7007f;"><b>DO NOT</b></span> modify any directories and file names.
{:.error}

## Submission Rules

Type your answer in the spaces provided in each file, labeled as `BEGIN ANSWER`.

1.  <span style="color:#f7007f;"><b>DO NOT</b></span> modify any `makefile`
2.  <span style="color:#f7007f;"><b>DO NOT</b></span> create more scripts other than what's given in the starter code
3.  <span style="color:#f7007f;"><b>DO NOT</b></span> modify any interface (keep original functions as-is)
4.  <span style="color:#f7007f;"><b>DO NOT</b></span> print <span style="color:#f7007f;"><b>anything</b></span> to the console, other than the provided print statements. Any print statements you used for debugging must be deleted.

Your shell should <span style="color:#f7007f;"><b>NOT</b></span> crash due to any input from the user or any <span style="color:#f7007f;"><b>ABSENCE</b></span> of input from the user after the given command. In other words, if the original code crash in any way even without you modifying it, we meant for you to fix it 🙂.
{:.error}

## Preparation

Go to `/pa1/` and type `make`. Then, run the compiled shell `./cseshell`. You should see all files compiled and the shell ran and _immediately_ returned as follows:
<img src="/50005-2023/assets/images/pa1/1.png"  class="center_seventy"/>

Notice how you have a few binaries available under `/bin`. Those are your <span style="color:#f77729;"><b>system programs</b></span>. We will execute these system programs later on from our shell.
<img src="/50005-2023/assets/images/pa1/2.png"  class="center_seventy"/>

## /bin/source

This directory contains the source file of your shell and also 7 <span style="color:#f77729;"><b>system programs</b></span>.
{:.warning}

### shell.h

Both files: `/bin/source/shell.c` and `/bin/source/shell.h` are the files containing the declaration and implementation for your shell program. The header file (shell.h) contains function declarations, imports, and macro definitions, while the .c file contains the <span style="color:#f77729;"><b>implementation</b></span> for these functions.

Open `/bin/source/shell.h`. The first few lines of `#includes and #define` are libraries and macro definitions.

Next, we have these <span style="color:#f77729;"><b>array</b></span> of pointers `builtin_commands` global constant that stores the <span style="color:#f77729;"><b>strings</b></span> of built-in commands that the user can type into the shell. These commands are <span style="color:#f7007f;"><b>implemented in the shell</b></span> (instead of system programs).

```java
/*
  List of builtin commands, followed by their corresponding functions.
 */
const char *builtin_commands[] = {
    "cd",    // calls shell_cd
    "help",  // calls shell_help
    "exit",  // calls shell_exit
    "usage", // calls shell_usage
};

```

Next, is the function declarations of the shell that you will implement in `shell.c`:

```java
/*
The fundamental functions of the shell interface
*/
char *read_line_stdin(void);            // TASK 1
char **tokenize_line_stdin(char *line); // TASK 2
int process_command(char **args);       // TASK 3
void main_loop(void);                   // TASK 4
```

<span style="color:#f7007f;"><b>DO NOT</b></span> modify ANY of these original functions declared in `shell.h`: not the arguments, not the names, nothing. Leave it as it is.
{:.error}

### .c files

Other .c files inside `/bin/source` contains the implementation of your system programs. You will only need to modify three system programs: `countline.c`, `summond.c`, and `checkdaemon.c`. The others are <span style="color:#f77729;"><b>already implemented for you</b></span>.

## Expected commands

The following commands are <span style="color:#f77729;"><b>implemented</b></span> within the shell, and <span style="color:#f7007f;"><b>are already done for you</b></span>:

```java
  cd
  help
  exit
  usage
```

By "done" it means that the functionality has been implemented for you but it does not mean that it will **immediately** work out of the box because you have **not** implemented the shell yet. For instance `usage` will not work because it relies on global variable `current_number_tokens` which you will need to instantiate later.
{:.info}

This shell also supports: `listdir`, `listdirall`, `summond`, `checkdaemon`, `find`, and `countline` via the <span style="color:#f77729;"><b>system programs</b></span> in `/pa1/bin/`.

AGAIN, <span style="color:#f7007f;"><b>DO NOT</b></span> modify any directories and file names.
{:.error}

## Your Task

You are to implement:

- Task 1-4: the <span style="color:#f77729;"><b>fundamental</b></span> functions inside `shell.c` (Task 1-4)
- Task 5-7: <span style="color:#f77729;"><b>three</b></span> system program: `countline` (Task 5), `summond` (Task 6), and `checkdaemon` (Task 7).
