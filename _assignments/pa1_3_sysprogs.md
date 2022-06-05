---
title: countline.c
permalink: /assignments/pa1_3_sysprogs
key: assignments-pa1_3_sysprogs
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

We have completed the shell thus far. Now, we will implement one system program: `countline`. Open `count_line.c` inside `/bin/source`.

## Task 5 (1%)
`Task 5:`{:.info} Complete the implementation of `countline` to print out the number of lines in a file. 

The `countline` system program accepts a pathname in `args[1]`, open that file, and stores the number of new lines in the file inside the variable `number_of_lines`. Complete the following function inside `count_line.c`:
```cpp
/*
Count the number of lines in a file
*/
int execute(char **args)
{

    int number_of_lines = 0;
    FILE *fp;

    // open file in read mode
    fp = fopen(args[1], "r");

    if (!fp)
    {
        printf("File %s cannot be found.\n", args[1]);
        return 1;
    }

    /** TASK 5  **/
    // ATTENTION: you need to implement this function from scratch and not to utilize other system program to do this
    // 1. Read the file line by line by using getline(&buffer, &size, fp)
    // 2. Loop, as long as getline() does not return -1, keeps reading and increment the count
    // 3. Store the count at number_of_lines
    // DO NOT PRINT ANYTHING TO THE OUTPUT

    /***** BEGIN ANSWER HERE *****/

    /*********************/
    fclose(fp); // close file.
    printf("%d \t %s \n", number_of_lines, args[1]);
    return 1;
}
```

### Test Task 5
Simply <span style="color:#f7007f;"><b>recompile</b></span> with `make`

The `countline` system program must work as follows. Look at these outputs <span style="color:#f77729;"><b>carefully</b></span>, and no, there's no typos there. 
<img src="/50005/assets/images/pa1/7.png"  class="center_full"/>

DO NOT print anything else in the console as part of your answer.
{:.error}

### Commit Task 5
Save your changes and commit the changes:

```
git add ./bin/source/count_line.c    
git commit -m "feat: Complete Task 5"
```