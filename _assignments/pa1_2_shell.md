---
title: shell.c
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
   - <span style="color:#f77729;"><b>Fetch</b></span> one line of user input from `stdin` using `read_line_stdin()` function
   - Then, pass the output of `read_line_stdin()` to `tokenize_line_stdin(char *line)` for parsing and tokenising (separated by space) user input command
   - Then, pass the output of `tokenize_line_stdin(char *line)` as the input to `process_command(char **args)`, where we <span style="color:#f77729;"><b>execute</b></span> the appropriate system program or builtin shell commands.
     Of course you need to code a way to <span style="color:#f77729;"><b>terminate</b></span> the shell, i,e: jumps out of the loop when a user type `exit` onto the terminal. This is done by calling `shell_exit` function that's already implemented for you.

DO NOT print anything else in the console as part of your answer.
{:.error}

# Task 1 (1%)

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

# Task 2 (2%)

`TASK 2:`{:.info} Implement `tokenize_line_stdin` in `shell.c`.

Complete the following function:

```cpp

/**
 Receives the *line, and return char** that tokenize the line
**/

char **tokenize_line_stdin(char *line)
{

  // create local variables to store the array of pointers to the first char of each word in the line
  int buf_size = SHELL_BUFFERSIZE, position = 0;     // assume there's also BUFFERSIZE amount of token, which is certainly enough because there's only BUFFERSIZE amount of chars
  char **tokens = malloc(buf_size * sizeof(char *)); // an array of pointers to the first char that marks a token in line
  char *token;

  /** TASK 2 **/
  // 1. Check that char ** that is returned by malloc is not NULL
  // 2. Tokenize the input *line using strtok() function
  // 3. Store the address to first letter of each word in the command in tokens
  // 4. Add NULL termination in tokens so we know how many "valid" addresses there are in tokens
  // DO NOT PRINT ANYTHING TO THE OUTPUT
  /***** BEGIN ANSWER HERE *****/

  /*********************/

  return tokens;
}
```

This function <span style="color:#f77729;"><b>receives</b></span> a <span style="color:#f77729;"><b>pointer</b></span> to the memory location that contains strings of character of the user input. It will return the pointers to addresses (`char**`) that tokenize the input.

For example, let’s say the user type in the following input:
`gcc shell.c -o cseshell`

This string of characters are stored within a <span style="color:#f77729;"><b>persistent</b></span> memory location. There are <span style="color:#f77729;"><b>four</b></span> tokens separated by spaces. These tokens are:

- gcc
- shell.c
- -o
- customshell

We want this function to return a <span style="color:#f77729;"><b>list of addresses</b></span> pointing to the <span style="color:#f77729;"><b>address</b></span> first letter of each word in the command (hence `char**` type). Read Part 3 of [C for Babies](https://docs.google.com/document/d/1hcMLXwKqblB9UtalLnbPjIHmxrdAnZaK7Haadhql9jo/edit#) if you don't know what `*` (pointer) is.

### strtok

You must use `strtok(char* input, delimiter)` to tokenize the input line. You can read on how to use it [here](http://man7.org/linux/man-pages/man3/strtok.3.html).

Note that for the delimiter field of `strtok`, remember to add in <span style="color:#f77729;"><b>other</b></span> delimiter as well such as \n and \t so that it will strip these trailing characters. This is what the macro `SHELL_INPUT_DELIM` for in `shell.h`.

### Test Task 2

Comment the `main()` function in `shell.c` and replace with the following:

```cpp
int main(int argc, char **argv)
{

 printf("Shell Run successful. Running now: \n");

 char* line = read_line_stdin();
 printf("The fetched line is : %s \n", line);

 char** args = tokenize_line_stdin(line);
 printf("The first token is %s \n", args[0]);
 printf("The second token is %s \n", args[1]);

 return 0;
}
```

Recompile and run. You should see that what you typed in the console will be printed back, and <span style="color:#f77729;"><b>tokenised</b></span> after you pressed <span style="color:#f7007f;"><b>enter</b></span>.

<img src="/50005/assets/images/pa1/4.png"  class="center_seventy"/>

### Commit Task 2

Save your changes and commit the changes:

```
git add ./bin/source/shell.c
git commit -m "feat: Complete Task 2"
```

# Task 3 (1%)

`TASK 3:`{:.info} Implement `process_command` in `shell.c`.

Complete the following function in `shell.c`:

```cpp

/**
   Call shell builtin functions if the command matches builtin_commands
   Otherwise, execute the system program
 */
int process_command(char **args)
{
  int child_exit_status;
  /** TASK 3 **/

  // 1. Check if args[0] is NULL. If it is, an empty command is entered, return 1
  // 2. Otherwise, check if args[0] is in any of our builtin_commands: cd, help, exit, or usage.
  // 3. If conditions in (2) are satisfied, call builtin shell commands, otherwise perform fork() to exec the system program. Check if fork() is successful.
  // 4. For the child process, call exec_sys_prog(args) to execute the matching system program. exec_sys_prog is already implemented for you.
  // 5. For the parent process, wait for the child process to complete and fetch the child's exit status value to child_exit_status
  // DO NOT PRINT ANYTHING TO THE OUTPUT

  /***** BEGIN ANSWER HERE *****/

  /*********************/
  if (child_exit_status != 1)
  {
    printf("Command %s has terminated abruptly.\n", args[0]);
  }
  return 1;
}
```

The parent process can read the exit status of the child process with:

```cpp
pid = fork();

if (pid > 0){
  int status;
  waitpid(pid, &status, WUNTRACED);
  // if child terminates properly, WIFEXITED(status) returns TRUE
  if (WIFEXITED(status)){
      child_exit_status = WEXITSTATUS(status);
  }
}
```

Note: `WEXITSTATUS(status)` returns the <span style="color:#f77729;"><b>exit status</b></span> of the child. This consists of the <span style="color:#f7007f;"><b>least significant 8 bits</b></span> of the status argument that the child specified in a call to exit(3) or as the argument for a return statement in `main()` in the child process. This is by nature, <span style="color:#f77729;"><b>limited in value</b></span> but good enough for our purpose to know whether the process as terminated normally.
{:.warning}

### Test Task 3

Comment the `main()` function in `shell.c` and replace with the following:

```cpp
int main(int argc, char **argv)
{

  printf("Shell Run successful. Running now: \n");

  char *line = read_line_stdin();
  printf("The fetched line is : %s \n", line);

  char **args = tokenize_line_stdin(line);
  printf("The first token is %s \n", args[0]);
  printf("The second token is %s \n", args[1]);

  // Setup path
  if (getcwd(output_file_path, sizeof(output_file_path)) != NULL)
  {
    printf("Current working dir: %s\n", output_file_path);
  }
  else
  {
    perror("getcwd() error, exiting now.");
    return 1;
  }
  process_command(args);

  return 0;
}
```

Recompile and run. You should see that what you typed in the console will be printed back, and <span style="color:#f77729;"><b>tokenised</b></span> and actually <span style="color:#f7007f;"><b>executed</b></span> after you pressed <span style="color:#f7007f;"><b>enter</b></span>.

<img src="/50005/assets/images/pa1/5.png"  class="center_seventy"/>

### Commit Task 3

Save your changes and commit the changes:

```
git add ./bin/source/shell.c
git commit -m "feat: Complete Task 3"
```

## Task 4 (2%)

`TASK 4:`{:.info} Implement `main_loop` in `shell.c`.

Complete the following function in `shell.c`:

```cpp
/**
  The main loop where one reads line,
  tokenize it, and then executes the command
 */
void main_loop(void)
{
  // instantiate local variables
  char *line;  // to accept the line of string from user
  char **args; // to tokenize them as arguments separated by spaces
  int status;  // if status == 1, prompt new user input. else, terminate the shell program.

  /** TASK 4 **/
  // write a loop where you do the following:
  // 1. invoke read_line_stdin() and store the output at line
  // 2. invoke tokenize_line_stdin(line) and store the output at args**
  // 3. execute the tokens using process_command(args)

  // Basic cleanup for the next loop
  // 4. free memory location containing the strings of characters
  // 5. free memory location containing char* to the first letter of each word in the input string
  // 6. check if process_command returns 1. If yes, loop back to Step 1 and prompt user with new input. Otherwise, exit the shell.
  // DO NOT PRINT ANYTHING TO THE OUTPUT
  do
  {
    // decorate the prompt
    time_t raw_time;
    struct tm *time_info;
    time(&raw_time);
    time_info = localtime(&raw_time);
    printf("🐚");
    yellow();
    char *timeString = asctime(time_info);
    timeString[strlen(timeString) - 1] = '\0';
    printf(" %s", timeString);
    red();
    printf(" CSEShell\n↳ ");
    reset();
    fflush(stdout); // clear the buffer and move the output to the console using fflush

    /***** BEGIN ANSWER HERE *****/
    status = shell_exit(args); // remove this line when you work on this task

    /*********************/
  } while (status);
}
```

Remove the line `status = shell_exit(args)` because that will cause your shell to terminate after one command. Follow the steps above by calling the functions you have implemented in Task 1-3.

### Test Task 4

You may use the original `main()` function given in the starter code to test. Your shell now should be able to prompt you for more commands after executing one.

<img src="/50005/assets/images/pa1/6.png"  class="center_seventy"/>

The shell should <span style="color:#f77729;"><b>never</b></span> crash no matter what input you give (empty lines, commands without arguments, etc). It should only terminate when you type `exit`.
{:.error}

### Commit Task 4

Save your changes and commit the changes:

```
git add ./bin/source/shell.c
git commit -m "feat: Complete Task 4"
```
