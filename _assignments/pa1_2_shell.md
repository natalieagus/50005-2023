---
title: Shell.c
permalink: /assignments/pa1_2_shell
key: assignments-pa1_2_shell
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

## Basics: How Shell Works
Read `shell.c` <span style="color:#f77729;"><b>carefully</b></span> before you start.
{:.warning}

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

