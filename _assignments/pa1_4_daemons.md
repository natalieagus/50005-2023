---
title: Daemons
permalink: /assignments/pa1_4_daemons
key: assignments-pa1_4_daemons
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

The program `summond.c` in `/bin/source` <span style="color:#f77729;"><b>summons</b></span> a daemon process and <span style="color:#f77729;"><b>terminates</b></span> so that the shell may continue to print the next prompt. This is unlike other programs where the shell waits for it to finish before printing the next prompt.

**Daemons** are processes that are often started when the system is bootstrapped and terminate only when the system is shut down. They <span style="color:#f77729;"><b>don’t have a controlling terminal</b></span> and they run in the <span style="color:#f77729;"><b>background</b></span>. In other words, a Daemon is a computer program that runs as a background process, rather than being under the direct control of an interactive user. UNIX systems have numerous daemons that perform day-to-day activities.
{:.warning}

Traditionally, the process names of a daemon end with the letter `d`, for clarification that the process is in fact a daemon, and for differentiation between a daemon and a normal computer program. For example: `/sbin/launchd`, `/usr/sbin/syslogd`, `/usr/libexec/configd`, etc (may vary from machine to machine). You can launch the command `ps -ef` to identify these processes whose names end with a suffix ‘d’:

<img src="/50005-2023/assets/images/pa1/8.png"  class="center_seventy"/>

For the sake of our lab and our machine’s health, our daemon <span style="color:#f77729;"><b>terminates</b></span> after a certain period of time and _violates the traditional daemon definition_, but we’re sure you get the idea.
{:.info}

### Overgoogling

The basic information about daemons presented in this handout is <span style="color:#f7007f;"><b>sufficient</b></span>. Do not over-Google about Daemons unless you are really interested in it. The concept of daemons alone is very complex and large, and is out of our scope.

## Characteristics of a Daemon Process

A daemon process is still a normal process, running in <span style="color:#f7007f;"><b>user mode</b></span> with certain characteristics which distinguish it from a normal process.

The characteristics of a daemon process are listed below.

### No controlling terminal

By definition, a daemon process <span style="color:#f7007f;"><b>does not require direct user interaction</b></span> and therefore must detach itself from any controlling terminal.
{:.warning}

In the `ps -ef` output, if the `TTY` column is listed as a `?` meaning it does not have a controlling terminal.

<img src="/50005-2023/assets/images/pa1/9.png"  class="center_seventy"/>

### PPID is 1

The PPID of a daemon process is 1, meaning that whoever was creating the daemon process must <span style="color:#f7007f;"><b>terminate</b></span> to let the daemon process be <span style="color:#f77729;"><b>adopted</b></span> by the `init` process (or equivalent).

**Note:** Although it is common, not all systems assign the `init` process to adopt orphaned process. More modern linux distros uses `systemd` process (or other designated descendant processes or equivalent) to adopt all orphaned process. The pid of `init` or `systemd` or equivalent processes is also **not always 1**. As long as your daemon process' ppid is the same as the pid of `init` _or_ pid of other special instances of `init` _or_ pid of `systemd` and equivalent, it is **acceptable** as long as it is **consistent** (that is the same designated system process is always adopting your daemon processes). When we test it in our system, we _know_ exactly what process will adopt your daemon process, so you don't need to worry.
{:.warning}

### Working directory: root

The working directory of the daemon process is typically the `root` (/) directory.

### Closes all uneeded fd

It closes all unneeded file descriptors. In <span style="color:#f77729;"><b>Unix</b></span> and related computer operating systems, a file descriptor (FD, less frequently fildes) is an abstract indicator (handle) used to access a file or other input/output resource, such as a pipe or network socket.

Also, it <span style="color:#f77729;"><b>closes</b></span> and <span style="color:#f77729;"><b>redirect</b></span> fd 0, 1, and 2 to `/dev/null`.
{:.warning}

### Logging

It logs messages through a <span style="color:#f77729;"><b>central</b></span> logging facilities: the `BSD syslog`.

## Task 6 (2%)

`TASK 6:`{:.info} Summon a daemon process in `summond.c`.

Complete the following function in `summond.c`:

```cpp
/*This function summons a daemon process out of the current process*/
static int create_daemon()
{

    /* TASK 6 */
    // Incantation on creating a daemon with fork() twice

    // 1. Fork() from the parent process
    // 2. Close parent with exit(1)
    // 3. On child process (this is intermediate process), call setsid() so that the child becomes session leader to lose the controlling TTY
    // 4. Ignore SIGCHLD, SIGHUP
    // 5. Fork() again, parent (the intermediate) process terminates
    // 6. Child process (the daemon) set new file permissions using umask(0). Daemon's PPID at this point is 1 (the init)
    // 7. Change working directory to root
    // 8. Close all open file descriptors using sysconf(_SC_OPEN_MAX) and redirect fd 0,1,2 to /dev/null
    // 9. Return to main
    // DO NOT PRINT ANYTHING TO THE OUTPUT
    /***** BEGIN ANSWER HERE *****/

    /*********************/

    return 0;
}
```

## Implementation notes

The steps written as a pseudocode above is a guide on how to create a daemon process that possesses the characteristics written previously. The sections below explains why each steps are <span style="color:#f77729;"><b>crucial</b></span>.

### Step 1: first fork

The fork() splits this process into <span style="color:#f77729;"><b>two</b></span>: the <span style="color:#f77729;"><b>parent</b></span> (process group leader) and the <span style="color:#f77729;"><b>child</b></span> process (that we will call <span style="color:#f7007f;"><b>intermediate</b></span> process for the next few sections).

The reason for this `fork()` is so that the parent returns immediately and our shell does not wait for the daemon to exit (because usually, daemons are background processes that do not exit until the system is shut down. We don’t want our shell to hang forever).

### Step 2: exit

At this point, the shell will return while the <span style="color:#f77729;"><b>intermediate process</b></span> proceed to spawn the daemon.

### Step 3: setsid

The child (<span style="color:#f77729;"><b>intermediate</b></span> process) process is by default not a process group leader. It calls `setsid()` to be the <span style="color:#f77729;"><b>session leader</b></span> and <span style="color:#f77729;"><b>loses controlling</b></span> TTY (terminal).

`setsid()` is only <span style="color:#f77729;"><b>effective</b></span> when called by a process that is not a process group leader. The `fork()` in step 1 ensures this. The system call `setsid()` is used to create a new session containing a single (new) process group, with the current process as both the session leader and the process group leader of that single process group. You can read more about setsid() [here](http://man7.org/linux/man-pages/man2/setsid.2.html).

#### Group and Session leader

Compile and try this code:

```cpp
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>


int main(){
   pid_t pid = fork();
   if (pid == 0){
       printf("Child process with pid %d, pgid %d, session id: %d\n", getpid(), getpgid(getpid()), getsid(getpid()));
       setsid(); // child tries setsid
       printf("Child process has setsid with pid %d, pgid %d, session id: %d\n", getpid(), getpgid(getpid()), getsid(getpid()));

   }
   else{
     printf("Parent process with pid %d, pgid %d, session id :%d\n", getpid(), getpgid(getpid()), getsid(getpid()));
       setsid(); // parent tries setsid
       printf("Parent process has setsid with pid %d, pgid %d, session id: %d\n", getpid(), getpgid(getpid()), getsid(getpid()));
       wait(NULL);

   }
   return 0;
}
```

It results in such output:

<img src="/50005-2023/assets/images/pa1/10.png"  class="center_seventy"/>

Let's analyse them <span style="color:#f77729;"><b>line by line</b></span>.

<span style="color:#f77729;"><b>The first line:</b></span> The parent process has pid == pgid, that is `75614`.

- This tells us that this process ‘iddemo’ is the process group leader, but <span style="color:#f77729;"><b>not</b></span> a session leader since the session id `27063` is not equal to the pid `75614`.

<span style="color:#f77729;"><b>The third line:</b></span> When process `75614` <span style="color:#f77729;"><b>forks</b></span>, it has a child process with pid `75615`. It is clear that since child `pid != pgid`, then the child process is <span style="color:#f7007f;"><b>not</b></span> a session leader and is not a group leader either.

So who is 27063? We can type the command ps -a -j and find a process with pid 27063. Apparently, it's the `zsh`, the shell itself, connected to the controlling terminal `s002`.

<img src="/50005-2023/assets/images/pa1/11.png"  class="center_seventy"/>

<span style="color:#f77729;"><b>The third and fourth lines:</b></span> When <span style="color:#f7007f;"><b>both</b></span> the child and parent process attempt to call `setsid`,

- In the child process, `setsid` effectively makes the `pgid` and session `id` to be equal to its pid, `75615`.
- In the parent process, setsid has <span style="color:#f7007f;"><b>no effect</b></span> on the session id, since the manual states that setsid only sets the process to be the session and process group leader if it is called by a process that is <span style="color:#f7007f;"><b>not</b></span> a process group leader.

Thanks to Step 1, the effect of `setsid` in Step 3 works as intended, and our <span style="color:#f77729;"><b>intermediate process</b></span> now lose the controlling terminal (part of a requirement to be a daemon process).
{:.warning}

### Step 4: Ignore SIGCHLD and SIGHUP

This intermediate process is going to `fork()` one more time in Step 5 to create the <span style="color:#f7007f;"><b>daemon</b></span> process.

By ignoring `SIGCHLD`, our daemon process -- the child of this intermediate process <span style="color:#f7007f;"><b>will NOT be a zombie process</b></span> when it terminates. Normally, a child process will be a zombie process if the parent does not `wait` for it.

- Since `SIGCHLD` is ignored, when the daemon (child of this intermediate process) exits, it is reaped immediately.
- However, the daemon will outlive the parent process anyway, so it does not really matter. This step is just for "in case".

Also, this intermediate process is a <span style="color:#f77729;"><b>session leader</b></span> (from step 3, since we need to <span style="color:#f77729;"><b>lose</b></span> the controlling terminal).

- If we terminate a session leader, a `SIGHUP` signal will be received and the children of the session leader will be <span style="color:#f77729;"><b>killed</b></span>.
- We do not want our daemon (child of this process) to be killed, therefore we need to call `signal(SIGHUP, SIG_IGN)` first <span style="color:#f77729;"><b>before forking</b></span> in Step 5.

#### Step 5: second fork

The child of this intermediate process is the <span style="color:#f7007f;"><b>daemon</b></span> process.
{:.error}

The second fork, is useful for allowing the parent process (intermediate process) to terminate. This ensures that the child process is <span style="color:#f77729;"><b>not a session leader</b></span>.

Why must you ensure that the daemon is not a session leader? Since a daemon has no controlling terminal, if a daemon is a session leader, an act of opening a terminal device will make that device the controlling terminal.
{:.warning}

We do not want this to happen with your daemon, so this second `fork()` handles this issue. As mentioned above, before forking it is necessary to ignore `SIGHUP`. This prevents the child from being killed when the parent (which is the session leader) dies.

#### Step 6: umask(0)

The daemon process must set all new files created by it to have `0777` permission using umask(0).
{:.warning}

Setting the `umask` to `0` means that newly created files or directories created will have all permissions set, so any file created by this daemon can be accessed by any other processes because we can’t directly control the daemon anymore.

A umask of zero will cause all files to be created as permission `0777` or <span style="color:#f7007f;"><b>world-RW & executable</b></span>. Some system sets the permission as `0666` by default instead of `0777` for security reasons, and we don't want this!

How does setting `umask(0)` lands you with `0777` file permission?

`0777` actually stands for <span style="color:#f77729;"><b>octal</b></span> `777`. In C, the first 0 indicates octal notation, and you can translate the rest in binary: `111 111 111`, which means we will have `- rwx rwx rwx`, equivalent to global RW and executable for the file. If we want to restrict permission of <span style="color:#f77729;"><b>write</b></span> to only the <span style="color:#f77729;"><b>owner</b></span>, we can set `umask(022)` -- equivalent to having permission `0755`, with the binary: 111 101 101, which translates to `- rwx r-x r-x`.
{:.info}

The manual for umask can be found [here](http://man7.org/linux/man-pages/man2/umask.2.html).

### Step 7: chdir to root directory

Change the <span style="color:#f77729;"><b>current</b></span> working directory to root using `chdir("/")`. If a daemon were to leave its current working directory unchanged then this would prevent the filesystem containing that directory from being <span style="color:#f7007f;"><b>unmounted</b></span> while the daemon was running. It is therefore good practice for daemons to change their working directory to a <span style="color:#f7007f;"><b>safe</b></span> location that will never be umounted, like root.

### Step 8: handle fd 0, 1, 2 and close all unused fds

Close all open file descriptors and redirect `stdin`, `stdout`, and `stderr` (fd 0, 1, and 2 by default in UNIX systems) to `/dev/null` so that it won’t <span style="color:#f7007f;"><b>reacquire</b></span> them again if you mistakenly attempt to output to `stdout` or read from `stdin`.

Once it is running a daemon should <span style="color:#f77729;"><b>NOT</b></span> read from or write to the terminal from which it was launched. The simplest and most effective way to ensure this is to <span style="color:#f77729;"><b>close</b></span> the file descriptors corresponding to `stdin`, `stdout` and `stderr`. These should then be reopened, either to `/dev/null`, or if preferred to some other location.

There are two reasons for not leaving them closed:

1. To prevent code that refers to these file descriptors from failing
2. To prevent the descriptors from being reused when we call open() from the daemon’s code.

To <span style="color:#f77729;"><b>close</b></span> all opened file descriptors, you need to <span style="color:#f77729;"><b>loop through existing file descriptors</b></span>, and re-attach the first 3 fd’s using `dup(0)`. Note that `open()` and `dup()` will assign the <span style="color:#f77729;"><b>smallest</b></span> available file descriptor, in this case that is 0, 1, and 2 in sequence. You will learn more about these stuffs in the last OS chapter.

```cpp
   /* Close all open file descriptors */
   int x;
   for (x = sysconf(_SC_OPEN_MAX); x>=0; x--)
   {
       close (x);
   }

   /*
   * Attach file descriptors 0, 1, and 2 to /dev/null. */
   fd0 = open("/dev/null", O_RDWR);
   fd1 = dup(0);
   fd2 = dup(0);
```

### Step 9

Finally, return to the `main` function, of which the next function `daemon_work()` is called. Our simple daemon does nothing but write some fake logs to `logfile.txt` inside `/pa1`.

### Syslog Facility

In practice, there exist daemon error-logging facilities. The BSD syslog facility has been widely used since `4.2BSD`. You can print these logs using `syslog()`. It has already been given to you in the `main` function:

```cpp
    /* Open the log file */
    openlog("summond", LOG_PID, LOG_DAEMON);
    syslog(LOG_NOTICE, "Daemon started.");
    closelog();
```

Depending on your machine, your syslog facility may vary. In macOS, the syslog facility is the `Console.app`. You can simply search the message or the process name that is <span style="color:#f77729;"><b>summond</b></span>. In Ubuntu, you can read `/var/log/syslog` and search the message containing `summond` with `grep`:

<img src="/50005-2023/assets/images/pa1/12.png"  class="center_seventy"/>

In practice, once you direct the log message here, you can tell the facility on how to forward it to your own logfile. This process is rather specific and out of our scope, therefore for the sake of the lab, we just assume that our daemon can directly write to our own logfile: `/pa1/logfile.txt`:
<img src="/50005-2023/assets/images/pa1/13.png"  class="center_seventy"/>

### Test Task 6

Simply <span style="color:#f7007f;"><b>recompile</b></span> with `make`.

Run your shell and type `summond` command. Then, check if `logfile.txt` has been created and that the daemon is printing to it periodically. Afterwards, get the `pid` of that `summond` process using the command `ps -efj | grep summond`. This outputs the <span style="color:#f77729;"><b>pid</b></span> of `summond`.

Then, execute `lsof -p [summond_pid]` to see that it's first 3 file descriptors (0u, 1u, 2u) are attached to `/dev/null`.

<img src="/50005-2023/assets/images/pa1/14.png"  class="center_seventy"/>

Notice also that `ppid` of `summond` is 1 (the <span style="color:#f77729;"><b>third column</b></span>) of the output of `ps -efj | grep summond`.
`summond` is <span style="color:#f77729;"><b>neither</b></span> a session leader nor a group leader since its `PGID (15160) != PID (15161)`.

### Commit Task 6

Save your changes and commit the changes:

```
git add ./bin/source/summond.c
git commit -m "feat: Complete Task 6"
```

## Task 7 (1%)

`TASK 7:`{:.info} Implement system program `checkdaemon`.

Open `check_daemon.c`, and complete the following `execute` function:

```cpp
/*  A program that prints how many summoned daemons are currently alive */
int execute()
{

   // Create a command that trawl through output of ps -efj and contains "summond"
   char *command = malloc(sizeof(char) * 256);
   sprintf(command, "ps -efj | grep summond  | grep -Ev 'tty|pts' > output.txt");

   int result = system(command);
   if (result == -1)
   {
      printf("Command %s fail to execute. Exiting now. \n", command);
      return 1;
   }

   free(command);

   int live_daemons = 0;
   FILE *fptr;

   /* TASK 7 */
   // 1. Open the file output.txt
   // 2. Fetch line by line using getline()
   // 3. Increase the daemon count whenever we encounter a line
   // 4. Store the count inside live_daemons
   // DO NOT PRINT ANYTHING TO THE OUTPUT

   /***** BEGIN ANSWER HERE *****/

   /*********************/
   if (live_daemons == 0)
      printf("No daemon is alive right now.\n");
   else
   {
      printf("Live daemons: %d\n", live_daemons);
   }

   fclose(fptr);

   return 1;
}
```

This is nothing more than calling the `ps` command, and counting the number of lines in the produced file `output.txt`. Since you run `checkdaemon` via `cseshell` with working directory `/pa1/`, then the file `output.txt` is created inside `/pa1/` directory.

You can reuse your system program `countline` here or write one from scratch.

### Test Task 7

Simply <span style="color:#f7007f;"><b>recompile</b></span> with `make`, and run your shell after summoning a few daemons. The command `checkdaemon` should print out exactly as follows:

<img src="/50005-2023/assets/images/pa1/15.png"  class="center_seventy"/>

### Commit Task 7

Save your changes and commit the changes:

```
git add ./bin/source/check_daemon.c
git commit -m "feat: Complete Task 7"
```

# Push everything to remote repo

Before the due date, ensure that you always push to your remote repository. Assuming you are using the `master` branch:

```
git push -u origin master
```

# Debugging

Note that all the steps provided to you above are _necessary_ but <span style="color:#f77729;"><b>not sufficient</b></span>, meaning that if you were to blindly follow it **step by step**, bugs are still bound to happen, although _minor_. For instance, the shell will run but you might find garbage values printed out when you simply press a bunch of `return` keys in the shell, `usage` not executed properly, and many other small bugs that does not make the shell perfect. You are supposed to <span style="color:#f7007f;"><b>figure these out</b></span> by yourself, that's why this assignment is <span style="color:#f7007f;"><b>graded</b></span> (10%) and it is set as a pair work.

# Submission

Go to our bot and type `/start`. Follow the instructions there. You're only required to submit the github remote repo link.

<span style="color:#f7007f;"><b>We will clone your repo for grading. Make sure you do not print any other messages other than the starter code, because we are running your shell with an autograder to check for compilability and plagiarism checker, before going through them manually. </b></span>
{:.error}

# Summary

If you have reached this stage, congratulations for completing programming assignment 1!.

We hope that you have appreciated some valuable things regarding:

- Purposes of `fork()` in shell programs
- Incantation steps of creating daemon processes, and the reason why these steps must be done
- A few of C standard functions: `system`, `getline`, `malloc`, `free`, `signal`, `chdir`, among many others and amp up your experience in reading documentations
- Compiling using `make` and using the CLI
- <span style="color:#f77729;"><b>Basic</b></span> knowledge about file descriptors, and file permissions

Remember, <span style="color:#f7007f;"><b>DO NOT</b></span> add any more print statements other than those in the starter code.
{:.error}
