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
* <span style="color:#f77729;"><b>Create</b></span> a shell and wait for user input
* Write several other <span style="color:#f77729;"><b>system programs</b></span> that can be invoked by the shell
* <span style="color:#f77729;"><b>Parse</b></span> user input and invoke `fork()` with the appropriate program
* Create a program that results in a <span style="color:#f77729;"><b>daemon</b></span> process 
* Use your shell to keep track of the state of your daemon processes 

You may complete this assignment in <span style="color:#f7007f;"><b>pairs</b></span>. Indicate your partner's name in the google sheet provided in our course handout. 
{:.info}

## Starter Code

Download the starter code into your preferred working directory:
`git clone https://github.com/natalieagus/pa1.git`

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
Go to `/pa1/` and type `make`. Then, run the compiled shell `./cseshell`. You should see all files compiled and the shell ran and *immediately* returned as follows:
<img src="/50005/assets/images/pa1/1.png"  class="center_seventy"/>

Notice how you have a few binaries available under `/bin`. Those are your <span style="color:#f77729;"><b>system programs</b></span>. We will execute these system programs later on from our shell.
<img src="/50005/assets/images/pa1/2.png"  class="center_seventy"/>


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

This shell also supports: `listdir`, `listdirall`, `summond`, `checkdaemon`, `find`, and `countline` via the <span style="color:#f77729;"><b>system programs</b></span> in `/pa1/bin/`. 

AGAIN, <span style="color:#f7007f;"><b>DO NOT</b></span> modify any directories and file names. 
{:.error}

## Your Task

You are to implement:
* Task 1-4: the <span style="color:#f77729;"><b>fundamental</b></span> functions inside `shell.c` (Task 1-4)
* Task 5-7: <span style="color:#f77729;"><b>three</b></span> system program: `countline` (Task 5), `summond` (Task 6), and `checkdaemon` (Task 7). 

# Basics: How Shell Works

The shell is expected to always prompt for user input and `execute` the system program whose name matches the command:
1. The `main()` function in `shell.c` invokes `main_loop()`
2. The function `main_loop()` continuously <span style="color:#f77729;"><b>loops</b></span> to:
   * <span style="color:#f77729;"><b>Fetch</b></span> one line of user input from `stdin` using `read_line_stdin()` function
   * Then, pass the output of `read_line_stdin()` to `tokenize_line_stdin(char *line)` for parsing and tokenising (separated by space) user input command
   * Then, pass the output of `tokenize_line_stdin(char *line)` as the input to `process_command(char **args)`, where we <span style="color:#f77729;"><b>execute</b></span> the appropriate system program or builtin shell commands.
Of course you need to code a way to <span style="color:#f77729;"><b>terminate</b></span> the shell, i,e: jumps out of the loop when a user type `exit` onto the terminal. This is done by calling `shell_exit` function that's already implemented for you.

## Task 1
`TASK 1:`{:.info} Implement `read_line_stdin` in `shell.c`.

Complete the following function:
```cpp
/**
   Read line from stdin, return a pointer to the array containing the command string entered by the user
 */
char *read_line_stdin(void)
{
  size_t buf_size = SHELL_BUFFERSIZE;           // size of the buffer
  char *line = malloc(sizeof(char) * buf_size); // allocate memory space for the line*
  /** TASK 1 **/
  // read one line from stdin using getline()
  // 1. Check that the char* returned by malloc is not NULL
  // 2. Fetch an entire line from input stream stdin using getline() function. getline() will store user input onto the memory location allocated in (1)
  // 3. Return the char*
  // DO NOT PRINT ANYTHING TO THE OUTPUT

  /***** BEGIN ANSWER HERE *****/

  /*********************/

  return line;
}
```

You need to use `malloc` to ensure that `char *line` is still in the memory even after `read_line_stdin` returns. Read Part 6 of [C for Babies](https://docs.google.com/document/d/1hcMLXwKqblB9UtalLnbPjIHmxrdAnZaK7Haadhql9jo/edit#) if you don't know what `malloc` is. 

### Test Task 1
Comment the `main()` function in `shell.c` and replace with the following:
```cpp
int main(int argc, char **argv)
{
 
 char* line = read_line_stdin();
 printf("The fetched line is : %s \n", line);
 
 return 0;
}
```
Recompile and run. You should see that what you typed in the console will be printed back after you pressed <span style="color:#f7007f;"><b>enter</b></span>. 

<img src="/50005/assets/images/pa1/3.png"  class="center_seventy"/>

