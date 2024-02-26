---
title: Introduction
permalink: /ns_labs/lab2_1-intro
key: ns-labs-lab2_1-intro
layout: article
nav_key: ns_labs
sidebar:
  nav: ns_labs
license: false
aside:
  toc: true
show_edit_on_github: false
show_date: false
---

In NS Module 3, we examined how the security properties of <span style="color:#f77729;"><b>confidentiality</b></span> and data <span style="color:#f77729;"><b>integrity</b></span> could be protected by using symmetric key cryptography and signed message digests. In this lab exercise, you will learn how to write a program that makes use of <span style="color:#f77729;"><b>3DES</b></span> for data encryption, <span style="color:#f77729;"><b>SHA256</b></span> for creating message digests and <span style="color:#f77729;"><b>RSA</b></span> for digital signing.

At the end of this lab exercise, you should be able to:

- Understand how symmetric key cryptography can be used to encrypt data and protect its confidentiality.
- Understand how _multiple_ blocks of data are handled using different block cipher modes and padding.
- Compare the different block cipher modes in terms of how they operate.
- Understand how hash functions can be used to create fixed-length message digests.
- Understand how public key cryptography (e.g., RSA algorithm) can be used to create digital signatures.
- Understand how to create message digest using hash functions (e.g., MD5, SHA-1, SHA-256, SHA-512, etc) and sign it using RSA to guarantee data integrity.

There are 3 parts of this lab:

1. Symmetric key encryption for a <span style="color:#f77729;"><b>text</b></span> file
2. Symmetric key encryption for an <span style="color:#f77729;"><b>image</b></span> file
3. Signed message <span style="color:#f77729;"><b>digests</b></span>

## System Requirements

The starter code provided to you is written in Python. You need at least <span style="color:#f7007f;"><b>Python 3.10</b></span> to complete this assignment and the [`cryptography`](https://pypi.org/project/cryptography/) module. We will use the Python `cryptography` module to write our program instead of implementing 3DES, RSA and SHA-256 directly. You will also need this module for your Programming Assignment 2, so take this lab as a precursor to the asssignment.

While you can develop in Python using any OS, this lab is tested to run on a <span style="color:#f7007f;"><b>POSIX-compliant OS</b></span> (path, etc is resolved) so it is not guaranteed that it will run on other OS. You need to fix OS-specific problems in the starter code by yourself.
{:.warning}

## Starter Code

Download the starter code:

```
git clone https://github.com/natalieagus/nslab2.git
```

This will result in a directory with the following structure:

```
nslab2
   |-original_files
      |-longtext.txt
      |-shorttext.txt
      |-SUTD.bmp
      |-triangle.bmp
   |-.gitignore
   |-1_encrypt_text.py
   |-2_encrypt_image.py
   |-3_sign_digest.py
   |-README.md
   |-requirements.txt

```

Then, cd to `nslab2` and install the required modules:

```
python3 -m pip install -r requirements.txt
```

In this lab, you're only required to modify all the 3 python files. You don't need to submit your code, and simply answer the questionnaire on eDimension as usual. There are areas labeled in these `.py` files as `TODO`, and these are your tasks.

## Test the Starter Code

Running the three `.py` files at this point should give you the following printouts stating that you havent implemented the relevant tasks:
<img src="/50005-2023/assets/images/nslab2/1.png"  class="center_seventy"/>

## Debug Notes

### Invalid Syntax

Some of you might encounter the error when running `python3 [starter-code].py`

```python
  match convert_bytes_to_int(read_bytes(client_socket, 8)):
        ^
      case 0:

SyntaxError: invalid syntax
```

That's because your `python3` <span style="color:#f7007f;"><b>is NOT aliased</b></span> to `python3.10` or that you don't have `python3.10` installed. <span style="color:#f7007f;"><b>Fix this on your own.</b></span> You're a CS major student. Not knowing how to install Python and manage its libraries is _a really really bad thing_; it's like as if the entire 50.002 and the first 6 weeks of CSE doesn't mean anything to you.

In this handout, we assume that `python3` is <span style="color:#f77729;"><b>always aliased</b></span> to `python3.10`.
{:.error}

That is, if you type `python3` in the terminal, you'll see at least version 3.10 printed out:
<img src="/50005-2023/assets/images/pa2/5.png"  class="center_seventy"/>

### Module Not Found

Some of you might encouter the error `ModuleNotFoundError: No module named ‘cryptography’`. You should know what you need to do by now as a CS student. If you have installed cryptography using `pip install cryptography`, but still suffer from this error, it simply means that the `pip` you used does not install to the path library of whatever `python3` version you are using right now. That is, you may have <span style="color:#f7007f;"><b>mixed</b></span> up `Python` and `pip` versions on your machine.

Assuming your `python3` is aliased to `python3.10`, then you can do the following as stated above, instead of `pip3 install -r requirements.txt` you copy pasted from somewhere.

```
python3 -m pip install -r requirements.txt
```
