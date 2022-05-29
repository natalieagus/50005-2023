---
title: The Fix
permalink: /os_labs/lab2_fix
key: lab2_fix-cli
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

## seteuid()

One of the ways to <span style="color:#f77729;"><b>fix</b></span> this TOCTOU bug is to add just <span style="color:#f77729;"><b>one line of instruction</b></span> after `access()` (before `fopen()` is called) to manually set the effective UID of the process as the actual UID of the process.

You can do this using the following system call: `seteuid(getuid());`

### Task 12 
`TASK 12:`{:.info} Modify `vulnerable_root_prog.c` to add that <span style="color:#f77729;"><b>one line</b></span> of code above right under:
```cpp
if (!access(fileName, W_OK))
{
   ...
``` 

Then:
1. <span style="color:#f77729;"><b>Login as root</b></span> and <span style="color:#f77729;"><b>recompile</b></span> with `make` inside `/FilesForRoot/`.
2. Login back as your original user account, and cd to `/User/` again, and run `exploit.sh`. 
3. <span style="color:#f77729;"><b>Observe</b></span> the result 
4. Press `ctrl+c` to cancel the script (yes, this change will cause step 2 to run in infinite loop).



## Disable SUID 

Of course another way is to <span style="color:#f77729;"><b>disable</b></span> the SUID bit of  `vulnerable_root_prog` altogether, however in practice sometimes this might not be ideal since there might be other parts of the program that requires execution with elevated privilege, <span style="color:#f77729;"><b>temporarily</b></span>. 

### Task 13 
`TASK 13:`{:.info} Run `exploit.sh` with `root_prog_nosuid`.

Open <span style="color:#f77729;"><b>exploit.sh</b></span> and replace `vulnerable_root_prog` with `root_prog_nosuid`, and run the script again (while logged in as user account). 

# Summary
Ensure that you have answered all questions in edimension corresponding to each task in this handout. No other separate code submission is needed. 

By the end of this lab, we hope that you have learned:
* What <span style="color:#f77729;"><b>SUID</b></span> bit does, and how can it be utilised to gain elevated privileges to access protected files 
* The differences between <span style="color:#f77729;"><b>root</b></span> and <span style="color:#f77729;"><b>normal</b></span> user 
* The meaning of file <span style="color:#f77729;"><b>permission</b></span>. Although we do not go through explicitly on how it is set, you can read about it [here](https://kb.iu.edu/d/abdb)  and experiment how to do it using the `chmod` command.  
* How <span style="color:#f77729;"><b>race condition</b></span> happens and how it can be used as an <span style="color:#f77729;"><b>attack</b></span> 
* How to <span style="color:#f77729;"><b>fix</b></span> the TOCTOU bug 


# TL;DR

1. Clone the repository:
```
git clone https://github.com/natalieagus/lab_toctou
```
2. Set root's password, and login as `root`:
```
sudo passwd root
su root 
```
3. Create other users (while logged in as root). Use a <span style="color:#f77729;"><b>good</b></span> password, like `LDcwzD&#6JKr`:
```
adduser test-user-0
adduser test-user-0 sudo
adduser test-user-1
adduser test-user-1 sudo
adduser test-user-2
adduser test-user-2 sudo
```
4. `make` inside `/FilesForRoot/` folder (while logged in as root):
```
cd FilesForRoot
make
```
5. Log in back to your regular user account:
```
su <username>
```
6. Change to `/User/` directory, `make`, then <span style="color:#f77729;"><b>exploit</b></span>:
```
cd ../User
make
./exploit.sh
```
7. Once `exploit.sh` succeeds, login to `test-user-0` with password `00000`. This proves that the attack has been successfully launched.
