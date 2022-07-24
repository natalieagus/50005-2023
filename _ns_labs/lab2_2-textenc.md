---
title: Symmetric Key Encryption on Text File
permalink: /ns_labs/lab2_2-sym-txt
key: ns-labs-lab2_2-sym-txt
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

Symmetric encryption is a type of encryption where only <span style="color:#f77729;"><b>one</b></span> key (a secret key) is used to both encrypt and decrypt a message. There are many symmetric key encryption algorithms: AES, 3DES, Blowfish, Rivest Cipher 5, etc. For this task, we are going to specifically use a 128-bit AES in CBC mode but we aren't going to build it from scratch. We will be using <span style="color:#f77729;"><b>Fernet</b></span>,  a recipe that provides symmetric encryption and authentication to data. It is a part of the `cryptography` library for Python, which is developed by the Python Cryptographic Authority (PYCA).

You can read more about [Fernet's full documentation](https://cryptography.io/en/latest/fernet/) and its [specs](https://github.com/fernet/spec/blob/master/Spec.md), but that's out of scope for this lab. We just want to use it as a tool to perform symmetric key encryption.

Open `1_encrypt_text.py` and let's begin. 

## Overview
There are two methods in the file, `enc_text(input_filename, output_filename)` and `dec_text(input_filename, output_filename)`. 
> Quickly study what it does.

If you scroll below, you'll find these methods called repeteadly to encrypt and decrypt various files:
```python
if __name__ == "__main__":
    enc_text("original_files/shorttext.txt", "output/enc_shorttext.txt")
    dec_text("output/enc_shorttext.txt", "output/dec_shorttext.txt")

    enc_text("original_files/longtext.txt", "output/enc_longtext.txt")
    dec_text("output/enc_longtext.txt", "output/dec_longtext.txt")
```

Our goal is to perform <span style="color:#f77729;"><b>symmetric key encryption</b></span> to plaintext files, and save it as encrypted files (with prefix `enc`). Then, take these encrypted files, decrypt it, and save it back to disk (with prefix `dec`).  

## Key Generation
### Task 1-1 
`TASK 1-1:`{:.info} Generate a fresh fernet key.

This key is used to both encrypt and decrypt messages. You need to <span style="color:#f77729;"><b>keep this safe</b></span> in real applications. Any adversaries that can get access to this key will be able to decrypt all your *supposedly confidential* messages. 

You can use the following method, and print it out to inspect its value. 
```python
symmetric_key = Fernet.generate_key()
print(symmetric_key)
```

## Cipher Instantiation
A <span style="color:#f77729;"><b>cipher</b></span> is name given in general that represents an algorithm for performing encryption or decryption. The *key* above is used as an input to the cipher to perform encryption or decryption.
### Task 1-2 
`TASK 1-2:`{:.info} Generate a cipher using Fernet (in both `enc` and `dec` functions)

You can take the symmetric key above and generate an instance of `Fernet` to help you `encrypt` and `decrypt` later:
```python
cipher = Fernet(symmetric_key)
```

## Encryption and Decryption
At this stage, encryption and decryption is *trivial*. You can encrypt data (in `bytes`) and decrypt data (in `bytes`) using the following method:
```python
message_bytes = b'Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna'
encrypted_message_bytes = cipher.encrypt(message_bytes)
decrypted_message_bytes = cipher.decrypt(encrypted_message_bytes)
assert message_bytes == decrypted_message_bytes
```

Now you should be able to complete these two tasks. 

### Task 1-3 
`TASK 1-3:`{:.info} Encrypt the raw bytes
Fill in your answer under the `TODO` for this task. 

### Task 1-4 
`TASK 1-4:`{:.info} Decrypt the raw bytes
Fill in your answer under the `TODO` for this task. 


## Creating a Printable Text Using base64
If you attempt to print the <span style="color:#f77729;"><b>encrypted</b></span> message bytes directly, you might see some weird characters printed out. That's because they're <span style="color:#f77729;"><b>no longer</b></span> represent any printable <span style="color:#f77729;"><b>encoding</b></span>. 

In `enc_text` function, we provide a way to <span style="color:#f77729;"><b>encode</b></span> the encrypted byte into a printable string using the `base64` module, before <span style="color:#f77729;"><b>saving</b></span> it to file. 

```python
   # Convert the ciphertext output to a printable string
   encrypted_base64_bytes = base64.b64encode(encrypted_bytes)
   encrypted_text = encrypted_base64_bytes.decode("utf8")

   # Save the printable string back to file
   with open(output_filename, "w") as fp:
      fp.write(encrypted_text)
```

This way you can open the `enc_[output_filename]` and read the encrypted bytes as proper string. 

As a result, when <span style="color:#f77729;"><b>decrypting</b></span> later on, we need to <span style="color:#f77729;"><b>decode</b></span> it back first before performing a decryption. We have done this for you in `dec_text` function:
```python
   # Open the file containing the cyphertext, read as string
   with open(input_filename, "r") as fp:
      encrypted_text = fp.read()

   # Convert the printable string back to bytearray
   encrypted_bytes = base64.b64decode(encrypted_text.encode("utf8"))
```

You may experiment writing the encrypted bytes to the file directly without encoding, and observe the difference.
{:.error}





