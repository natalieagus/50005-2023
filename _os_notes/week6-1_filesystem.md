---
title: Basic Concepts of File
permalink: /os_notes/week6-1_filesystem
key: os-notes-week6-1_filesystem
layout: article
nav_key: os_notes
sidebar:
   nav: os_notes
license: false
aside:
   toc: true
show_edit_on_github: false
show_date: false
---

A file is a <span style="color:#f77729;"><b>named</b></span> collection of related information that is stored on the secondary storage. It acts as a <span style="color:#f77729;"><b>logical</b></span> storage unit in the computer, defined by the operating system. In layman terms, they are a <span style="color:#f77729;"><b>group of data bytes</b></span> which are stored neatly in a known location with a unique name (<span style="color:#f77729;"><b>path</b></span>). 
{:.warning}

Files are mapped by the operating system onto physical devices that are non-volatile. Sometimes they are memory-cached for performance. 

<span style="color:#f77729;"><b>Volatile</b></span> storage devices are unable to store any data after the power source is removed. 
* Physical memory (RAM) is generally volatile. 
* SSDs, HDDs, and thumbdrives are non-volatile.

In this chapter, we will specifically learn about UNIX File system. 

There are <span style="color:#f77729;"><b>two</b></span> types of file: regular files and directories. More information can be found in the later parts. 
* <span style="color:#f77729;"><b>Directories</b></span>: 
  * Has a structured namespace, e.g: /users/Desktop 
* <span style="color:#f77729;"><b>Regular files</b></span>:
  * Can represent programs, or data
  * Can take any format: .txt, .bin, etc 
  * Can contain any information defined by its creator


## File Example
Consider the file with name: `Banker.c` below. 
1. The file consists of file <span style="color:#f77729;"><b>attributes</b></span> and file <span style="color:#f77729;"><b>content</b></span>. 
2. File attributes (stored in *inode*) contain <span style="color:#f77729;"><b>important</b></span> metadata such as name, size, datetime of creation, user ID, etc.  
3. Its <span style="color:#f77729;"><b>content</b></span> is a group of data bytes (~12KB) containing instructions to the Banker's algorithm written in C.

<img src="/50005/assets/images/week6/1.png"  class="center_seventy"/> 

When we use this file, we *don’t care* about its physical address (where it is actually stored on disk). We only care about its <span style="color:#f77729;"><b>path</b></span>. The path is a “logical” storage unit. 
{:.warning}

## Content Structure
File content structure that can exist in your computer varies greatly:
* None: **uninterpreted** sequence of bytes
* Simple:
  * **Lines** (text files)
  * Fixed Length Records (structured database, e.g: NRIC)
  * Variable Length Records (unstructured database, e.g: audio, video, books, are all of variable length)
* Complex Structure: 
  * **Executable** files (`ELF` files)   

## File Interpreters
Who can read and understand the content of the file? 
* The OS Kernel:
  * UNIX-based OS implements <span style="color:#f77729;"><b>directories</b></span> as special files
  * It knows the <span style="color:#f77729;"><b>difference</b></span> between directory & regular files; 
  * It supports <span style="color:#f77729;"><b>executable</b></span> files 
  * Doesn’t otherwise require/interpret any formats of other regular files
* System Programs:
  * <span style="color:#f77729;"><b>Linker</b></span> or <span style="color:#f77729;"><b>loader</b></span> that knows `ELF` formatted files 
  * <span style="color:#f77729;"><b>Compilers</b></span> that know certain file script formats (e.g: `.c` for `gcc`)
* User/Application Programs:
  * Web browser understands HTML, web-assembly format
  * Video players understands .mp4 format
  * PDF readers understands .pdf format
  * Excel understands .xls format, along with .csv 

### File Extension vs File Format
A file extension is the <span style="color:#f77729;"><b>character</b></span> or <span style="color:#f77729;"><b>group</b></span> of characters after the period that makes up an entire file name. It often indicate the file type, or file format, of the file, but <span style="color:#f7007f;"><b>not always</b></span>. We usually add it with a dot in the file name, e.g: `grades.xls`
* The file extension helps an OS to determine which program on your computer the file is <span style="color:#f77729;"><b>associated</b></span> with.
* For example, files ending with .xls will be associated with Excel -- opening it will open Excel and *load* that file into Excel. 

 Any file's extension can be renamed, but that <span style="color:#f7007f;"><b>won't convert</b></span> the file to another <span style="color:#f7007f;"><b>format</b></span> or change anything about the file <span style="color:#f7007f;"><b>other than this portion of its name</b></span>.
 {:.error}

 Hence, a file with a PDF <span style="color:#f7007f;"><b>format</b></span>  will NOT be converted into a ZIP format by simply changing it's name from `submission.pdf` to `submission.zip`. You will need a *tool* (e.g: the `zip` system program) to **convert** its format into an actual `ZIP` formatted file.
 
 Another example: a file named `grades.csv` contains an extension that is appropriately related to its actual format: comma separated values. A computer user could rename that file to `grades.mp3`, however that wouldn't mean you could play the file as some sort of audio on a media player. The file itself is still rows of text (a CSV file), not a compressed musical recording (an MP3 file).


