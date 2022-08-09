---
title: The Confidentiality Protocol
permalink: /assignments/pa2_3_cp
key: assignments-pa2_3_cp
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

<span style="color:#f77729;"><b>Congratulations!</b></span> 🎉,  you can now be assured that you are uploading your file to the <span style="color:#f77729;"><b>right</b></span> destination and not a fake, malicious server. 

However, can you trust the network path used for your upload? 
* It may go through many <span style="color:#f77729;"><b>intermediate</b></span> routers and communication links that you don’t know very well (or not at all). 
* Could people tap the links and <span style="color:#f77729;"><b>steal</b></span> your data? 
 
Unfortunately, <span style="color:#f7007f;"><b>yes</b></span>. Malicious parties can eavesdrop on your conversation with the server. Although *sometimes* confidentiality is not an issue, it is obviously not the case when you're uploading *personal* files to a remote server. 
 
To avoid the theft of data in transmission, you should implement a confidentiality protocol (CP). There are two basic ways to do this:
1. CP 1: Using <span style="color:#f77729;"><b>public</b></span> key cryptography
2. CP 2: Using <span style="color:#f77729;"><b>symmetric</b></span> key cryptography

## Confidentiality Protocol 1
### Task 2 (2%)
`TASK 2:`{:.info} Design a protocol to protect the confidentiality of the <span style="color:#f77729;"><b>content</b></span> of the uploaded file using <span style="color:#f77729;"><b>public key</b></span> cryptography. 

> For simplicity, the filename doesn't have to be encrypted. 

The suggested steps are as follows:
1. The client <span style="color:#f77729;"><b>encrypts</b></span> the file data (in units of <span style="color:#f77729;"><b>byte</b></span> blocks) before sending, and SecureStore <span style="color:#f77729;"><b>decrypts</b></span> on receive. 
    * Since the RSA key size is 1024 bits, it can encrypt/decrypt at most 128 bytes of data at a time. 
    * However you will need to use a <span style="color:#f77729;"><b>padding</b></span> for the data blocks and padding + data blocks must be 128 bytes at maximum.
    * There are two options:
      * `PKCS5`: min 11 bytes of padding, max 117 bytes data blocks
      * `OAEP` with `MGF` and `SHA-256`: min 66 bytes of padding, max 62 bytes data blocks

2. Since you were able to implement AP, you already have everything you need for CP1. Just remember that in public key cryptography, we could use <span style="color:#f77729;"><b>either</b></span> the public or private key for the encryption! 

Figure out <span style="color:#f77729;"><b>which key</b></span> to use to encrypt the data from the Client to securely upload your files to the server while protecting the <span style="color:#f77729;"><b>confidentiality</b></span> of the file.
{:.warning}


### Expanding the FTP 
Make a <span style="color:#f77729;"><b>copy</b></span> of Task 1 files: `ClientWithSecurityAP.py` and `ClientWithSecurityAP.py` each, and name it as `ClientWithSecurityCP1.py` and `ServerWithSecurityCP1.py`.
> Leave the original files as is. You should have 6 `.py` files now under `source/`.

There's no need to implement any new `MODE` in this task. 

Here's a recap of the `MODE` for CP1:
- `0`: client will send `M1`: size of filename, and `M2`: the filename (no need to modify, don't need to encrypt this)
- `1`: client will send `M1`: size of data block in `M2`, and `M2`: <span style="color:#f7007f;"><b>encrypted</b></span> blocks of data to the server. The server has to decrypt it before writing the file to `source/recv_files/` (NEW for Task 2)
  - Note: It is up to you to <span style="color:#f77729;"><b>either</b></span> concatenate ALL encrypted bytes and send it all at once as `M2` with the appropriate `M1` value (total encrypted bytes) *or* to send chunks of 128 bytes at repeatedly, e.g: send `M1`: 128 (as bytes), then `M2`: encrypted data chunk, repeatedly until the entire encrypted files are sent. The latter requires lots of socket `send` function calls, but it's your call. We have accounted for either cases in our grading scheme.  
  - No matter which method you choose, both are still *under* `MODE 1`, which means to send the file *data* in its entirety. 
  - Do not repeatedly send `MODE 1` for every 128-bytes chunk. 
- `2`: client closes connection (no need to modify, same as original)
- `3`: client begins authentication protocol as per Task 1 (no need to modify, same as AP in Task 1)  

And here's a little diagram to help you in your implementation. Remember, the little <span style="color:#f77729;"><b>vulnerability</b></span> highlighted in the AP section is still illustrated here. 
<img src="/50005/assets/images/pa2/6.png"  class="center_seventy"/>

You might want to refer to the [fifth page](https://natalieagus.github.io/50005/assignments/pa2_5_crypto) of this assignment to get started with some basic functionalities of Python `cryptography` module. 

### Grading
We will <span style="color:#f77729;"><b>manually</b></span> check the implementation of your `MODE 1` in both Client and Server scripts.

### Commit Task 2
Save your changes and commit the changes (assuming your current working directory is `source/`):

```
git add ServerWithSecurityCP1.py ClientWithSecurityCP1.py 
git commit -m "feat: Complete Task 2"
```


## Confidentiality Protocol 2
### Task 3 (2%)
`TASK 3:`{:.info} Design a protocol to protect the confidentiality of the <span style="color:#f77729;"><b>content</b></span> of the uploaded file using <span style="color:#f7007f;"><b>symmetric key</b></span> cryptography. 

Although CP1 is easy to implement, it is unbearably <span style="color:#f7007f;"><b>slow</b></span>. Try using it on a large file (>100MB, e.g `cmos.mp4`) and observe its slowdown relative to no encryption (no confidentiality). 

Hence, you will also implement this alternate confidentiality protocol that we call CP2:
* CP2 negotiates a <span style="color:#f77729;"><b>shared session key</b></span> (symmetric key) between the client and server, and 
> You <span style="color:#f7007f;"><b>should</b></span> use [`Fernet`](https://cryptography.io/en/latest/fernet/) to generate the session keys. Fernet is a symmetric key encryption system based on AES in CBC mode with a 128-bit key for encryption with PCKS7 padding. It is part of Python `cryptography` module). Recall that symmetric key encryption is <span style="color:#f77729;"><b>much</b></span> faster than RSA. 
* Uses the session key to provide <span style="color:#f77729;"><b>confidentiality</b></span> of the file data thereafter. 
 
You might want to refer to the [fifth page](https://natalieagus.github.io/50005/assignments/pa2_5_crypto) of this assignment to get started with some basic functionalities of Python `cryptography` module. 

Before you proceed, you might want to ask yourself: <span style="color:#f77729;"><b>who</b></span> should generate the session key? Client or server?
{:error}

### Expanding the FTP
Make a <span style="color:#f77729;"><b>copy</b></span> of Task 1 files: `ClientWithSecurityCP1.py` and `ServerWithSecurityCP1.py` each, and name it as `ClientWithSecurityCP2.py` and `ServerWithSecurityCP2.py`.
> Leave the original files as is. You should have 8 `.py` files now under `/pa2`.

Now both client and server must implement `MODE: 4`, which signifies the `key generation` handshake. 
1. Upon successful AP, client must then generate a session key using `Fernet`, then send `MODE: 4`, (`int` converted to `bytes`) to the server, followed by two messages:
   * `M1`: size of `M2` in bytes
   * `M2`: generated <span style="color:#f77729;"><b>session key bytes</b></span> that is encrypted (by what key? Remember we want to protect the confidentiality of this session key -- so only our server can decrypt it).
2. Then the client proceed to prompt the user with filename as per the regular FTP. 
3. Upon receiving `MODE: 4`, the server must receive and decrypt the session key. 

It goes without saying that `MODE: 1` implementation must be modified for both Server and Client to utilise the <span style="color:#f77729;"><b>session key</b></span> instead. 

Here's a recap of the `MODE` for CP2:
- `0`: client will send `M1`: size of filename, and `M2`: the filename (no need to modify, don't need to encrypt this)
- `1`: client will send `M1`: size of data block in `M2`, and `M2`: <span style="color:#f7007f;"><b>encrypted</b></span> blocks of data using the <span style="color:#f77729;"><b>session key</b></span>. The server has to decrypt it with the <span style="color:#f77729;"><b>session key</b></span> as well before writing it to file (NEW for Task 3)
- `2`: client closes connection (no need to modify, same as original)
- `3`: client begins authentication protocol as per Task 1 (no need to modify, same as AP in Task 1)  
- `4`: client begins sharing of session key protocol (NEW for Task 3)

And here's a little diagram to help you in your implementation. Remember, the little <span style="color:#f77729;"><b>vulnerability</b></span> highlighted in the AP section is still illustrated here. 
<img src="/50005/assets/images/pa2/7.png"  class="center_seventy"/>

### Grading
We will <span style="color:#f77729;"><b>manually</b></span> check the implementation of your `MODE 1` and `MODE 4` in both Client and Server scripts.

### Commit Task 3
Save your changes and commit the changes (assuming your current working directory is `source/`):

```
git add ServerWithSecurityCP2.py ClientWithSecurityCP2.py 
git commit -m "feat: Complete Task 3"
```

By the end of Task 3, you have 1 server-client script pairs for each task (8 scripts in total under `source/`): the original (non-secure), AP, CP1, and CP2 pairs. 

