---
title: The Authentication Handshake Protocol
permalink: /assignments/pa2_2_ap
key: assignments-pa2_2_ap
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

Imagine there exist a secure server called <span style="color:#f77729;"><b>SecureStore</b></span> (the server's name) at some _advertised_ IP address. As clients, we want to upload our files securely to these file servers. In the real world, this secure server might be Dropbox, Google Drive, etc. We can’t simply _trust_ the IP address because it’s easy to <span style="color:#f77729;"><b>spoof</b></span> IP addresses.

Hence, before the file upload, your client should <span style="color:#f77729;"><b>authenticate</b></span> SecureStore’s identity. To do that, you’ll implement an <span style="color:#f77729;"><b>authentication protocol</b></span> (AP) which bootstraps trust by a <span style="color:#f77729;"><b>certificate authority</b></span> (CA).

## Authentication Protocol Design

It is conceptually simple to design AP using <span style="color:#f77729;"><b>public</b></span> key (i.e., asymmetric) cryptography, that is to _authenticate_ that a message sent by SecureStore is _indeed_ created by it.

What you can do is:

1. Ask SecureStore to <span style="color:#f77729;"><b>sign</b></span> a message using its <span style="color:#f77729;"><b>private</b></span> key and send that message to you.
2. You can then use SecureStore's public key to <span style="color:#f77729;"><b>verify</b></span> the signed message.
3. If the check goes through, then since only SecureStore knows its private key but no one else, you know that the message **must** have been created by SecureStore.

> There’s one <span style="color:#f77729;"><b>catch</b></span>, however.

How can you obtain SecureStore’s public key <span style="color:#f77729;"><b>reliably</b></span>?

- If you simply ask SecureStore to send you the key, you’ll have to ensure that you’re _indeed_ receiving a reply _from_ SecureStore, otherwise <span style="color:#f7007f;"><b>a man in the middle attack</b></span> is possible like we learned in class.
- There's <span style="color:#f77729;"><b>circular dependency here</b></span>: you're replacing an authentication problem by another authentication problem!

## Certificate Authority

In the Internet, <span style="color:#f77729;"><b>trust</b></span> for public keys is bootstrapped by users going to well known providers (e.g., a <span style="color:#f77729;"><b>reputable</b></span> company like Verisign or a <span style="color:#f77729;"><b>government</b></span> authority like IDA) and <span style="color:#f7007f;"><b>registering</b></span> their public keys.

The registration process is supposed to be carefully scrutinized to ensure its credibility, e.g:

- You may have to provide elaborate <span style="color:#f7007f;"><b>documents</b></span> of your identity or
- <span style="color:#f7007f;"><b>Visit</b></span> the registration office personally so that they can interview you,
- <span style="color:#f7007f;"><b>Verify</b></span> your signature, etc (analogous to the process of opening an account with a local bank).

That way, Verisign or IDA can sign an entity’s (in our case, SecureStore’s) public key before giving it to you (client) and <span style="color:#f7007f;"><b>vouch</b></span> for its truthfulness.
{:.warning}

Note that we’re <span style="color:#f7007f;"><b>bootstrapping trust</b></span> because we’re replacing trust for SecureStore by trust for VeriSign or IDA. This works because it’s supposedly much easier for you to <span style="color:#f7007f;"><b>keep track</b></span> of information belonging to IDA (i.e., their public keys could be considered “common knowledge” that your browser can obtain) than information about a myriad of companies that you do business with.

### Getting CA's Root Certificate

A _certificate_ contains a public key. If we have the CA's certificate, we have its public key too.

> How do we get CA's information then? How do we know that CA's keys _indeed_ belong to the CA?

Either your browser or your OS <span style="color:#f77729;"><b>ships</b></span> with <span style="color:#f77729;"><b>a set of trusted CA certificates</b></span>. If a new CA comes into the market, then the browser or the OS will need to ship a software updates containing the new CA's information.

### csertificate

In this assignment, you won’t use VeriSign or IDA. Instead, the CSE teaching staff has volunteered to be your trusted CA -- we call our service <span style="color:#f7007f;"><b>csertificate</b></span>, and we’ll tell you (i.e., your SecureStore and any client programs) our public key in <span style="color:#f77729;"><b>advance</b></span> as “common knowledge”.

We have given our CA certificate in your starter project: `source/auth/cacsertificate.crt`.

> This is analogous to how your browser or OS ships with a set of trusted CA certificates.

## Proposed AP

<span style="color:#f7007f;"><b>Please read this whole section from start to end carefully before you begin your implementation</b></span>. You need to stick to the suggested <span style="color:#f77729;"><b>protocol</b></span> carefully.

### Task 1 (3%)

`TASK 1:`{:.info} Implement an Authentication Protocol. You are expected to make a copy of the starter non-secure FTP and modify it. Read along first to get the general idea.

To get you started, we suggest a few things that you can do to authenticate the identity of SecureStore:

1. SecureStore **generates** RSA private and public key pair (use <span style="color:#f77729;"><b>1024 bit</b></span> keys). It also creates a certificate signing request (`.csr`), where it submits the public key and other credentials (e.g., its legal name).

   - You can already do this by running `python3 generate_keys.py` file provided to you in `source/auth`.
   - <span style="color:#f77729;"><b>Change</b></span> your working directory to `source/auth` first before running `generate_keys.py`.
   - Study what `generate_keys.py` does carefully.

2. SecureStore **uploads** the certificate signing request (`.csr` file) for approval to csertificate, our CA. **Our CA is integrated with our bot**. <span style="color:#f77729;"><b>Simply type `/start` to our bot and follow the instructions.</b></span> Our CA will:

   - <span style="color:#f77729;"><b>Verify</b></span> the request, and
   - Sign it to create a <span style="color:#f77729;"><b>certificate</b></span> containing SecureStore's public key (`server_signed.crt`), and
   - Pass the signed certificate back to SecureStore.
   - This certificate is now bound to SecureStore and contains its public key.

3. SecureStore retrieves the signed certificate by our CA, and you <span style="color:#f77729;"><b>MUST save the file `server_signed.crt` under `source/auth/` directory.</b></span> Do not change it to any other name.

4. When people (e.g., a client program) later ask the Server for its public key, it provides this <span style="color:#f77729;"><b>signed</b></span> certificate.
   - The client can <span style="color:#f77729;"><b>verify</b></span> that this certificate is indeed <span style="color:#f77729;"><b>signed</b></span> (authorised) by our trusted CA csertificate using csertificate's Public Key, embedded within `cassertificate.crt`.
   - Recall that csertificate's public key is embedded within `cacsertificate.crt` given to you in the starter code. In real life, this should be a _well known_ certificate shipped by the Browser/OS.
   - However for the sake of this assignment, we are simply "pretending" to be a legitimate CA but our certificate is as good as self-signed.

  <img src="/50005-2023/assets/images/pa2/img.jpg"  class="center_fifty"/>

#### Authentication Protocol Basis

The diagram below gives the basis of a possible authentication protocol. Take note of the `MODE`. The `MODE` preceeding the streams of messages (`M1, M2`) sent by the Client tells the server what _kind_ of message it is. Then, `M1` tells the server the size of `M2` so that the Server can `read` that appropriate number of bytes from the <span style="color:#f77729;"><b>socket</b></span>.

> Socket `read` is a <span style="color:#f77729;"><b>blocking</b></span> operation, thus it is important to design a <span style="color:#f77729;"><b>protocol</b></span> between client and server properly to know who is supposed to read/send at any given time.

The size of `M1` is designed to be not more than `8` bytes. The next section will explain this in more detail.

<img src="/50005-2023/assets/images/pa2/1.png"  class="center_seventy"/>

The server (SecureStore) has the following items to encrypt/decrypt messages from the client when necessary:

- Signed certificate `server_signed.crt`, containing server public key: $$Ks^+$$ signed by our CA
- Server private key: $$Ks^-$$, can be extracted from `server_private_key.pem`

Upon <span style="color:#f77729;"><b>successful</b></span> connection, the client shall send an arbitrary `message` to the server, and the server <span style="color:#f77729;"><b>must</b></span> <span style="color:#f7007f;"><b>sign</b></span> it by its private key. Then, the server sends both the signed message + `server_signed.crt` certificate to the client.

During the step labeled `CHECK SERVER ID`, the client must:

1. <span style="color:#f77729;"><b>Verify</b></span> the signed certificate sent by the Server using ca's public key $$Kca^+$$ obtained from `cacsertificate.crt` file,
2. Extract `server_public_key`: $$Ks^+$$ from it,
3. Decrypt signed message: $$Ks^-\{M\}$$ (using the `verify` method) to verify that $$M$$ is the same message sent by the client in the first place
4. Check <span style="color:#f77729;"><b>validity</b></span> of server cert:
   - Afterwards, you can <span style="color:#f77729;"><b>check</b></span> server certificate's <span style="color:#f77729;"><b>validity</b></span> as well
   - You can even do this _before_ Step 1-3 above, it is up to you
   - We can't explicitly test you on this without spending too much manpower, but this is _good practice_
5. If the `CHECK` passes, the client will proceed with file upload protocol (next task)
6. In the event that the `CHECK` fails, the client must <span style="color:#f77729;"><b>close</b></span> the connection immediately (abort mission)

However, theres one <span style="color:#f7007f;"><b>problem</b></span> with the proposed AP above (means it has a <span style="color:#f77729;"><b>vulnerability</b></span>). You need to <span style="color:#f77729;"><b>identify and fix this problem</b></span> in your submission.
{:.error}

> Hint: read back the <span style="color:#f77729;"><b>second</b></span> requirement of the assignment in the previous page.

A complete implementation of the AP will grant you 3% of your grades.

### Starter Non-secure FTP

The <span style="color:#f7007f;"><b>starter code</b></span> given to you has implemented a simple, <span style="color:#f7007f;"><b>non secure</b></span> file transfer protocol (FTP).

1. Upon successful `connect`, the Client will `send` messages to the server in this general form:
   1. First to send a `MODE` (an `int` converted to bytes),
   2. Then to send streams `messages`. There might be more than 1 message sent after each `MODE` depending on the purpose of the `MODE`.
      - In the diagram above, the first message is labeled as `M1`: the size of the second message (this size alone not exceeding 8 bytes to represent by design),
      - And then the second message labeled as `M2`: the actual content of the message.
        > Note that sockets always send and receive in <span style="color:#f77729;"><b>bytes</b></span>, so whatever you send and read through sockets **must** be in terms of bytes.
      - <span style="color:#f7007f;"><b>This is so that the server is able to `read` from the socket exactly those `N` number of bytes dictated in the first message. </b></span>
2. The Server will at first listen to message `MODE` sent by client at all times, and then attempt to `read` accordingly.
3. There are 3 default `MODE` provided, which upon received, the server will categorise the second message as the following:

   - `0`: two messages are expected following a `0`,
     - `M1`: The filename length (this is no more than 8 bytes long <span style="color:#f77729;"><b>by design</b></span>)
       > Note that it does NOT mean that the length of the filename is less than `8`. It means that the length of the filename needs at most 8 bytes to represent. Thats covering integer value between 0 to $$2^{8\times8}$$ for unsigned integers, that's a large number!
     - `M2`: The filename data itself with size dictated in the first message
   - `1`: two messages are expected following a `1`,

     - `M1`: <span style="color:#f77729;"><b>size</b></span> of the data block in bytes (not more than 8 bytes long <span style="color:#f77729;"><b>by design</b></span>)
     - `M2`: the data block itself with size dictated in the first message

   - `2`: no message is expected. This is a <span style="color:#f77729;"><b>connection close</b></span> request.

> Note that it is <span style="color:#f77729;"><b>not always</b></span> necessary to send two messages after each `MODE` `int` data is sent, if the server knows <span style="color:#f77729;"><b>EXACTLY</b></span> the number of bytes that the client will send following a `MODE`. For instance, if <span style="color:#f77729;"><b>filename</b></span> has a <span style="color:#f77729;"><b>fixed</b></span> byte length: 32 bytes for instance, we can condense `MODE 0` in the server side to always do a `read_bytes(client_socket, 32)`. For this assignment however, follow the protocol stated in this handout closely.

### Expanding the FTP

Make a <span style="color:#f77729;"><b>copy</b></span> of the original `ClientWithoutSecurity.py` and `ServerWithoutSecurity.py` each, and name it as `ClientWithSecurityAP.py` and `ServerWithSecurityAP.py`.

> Leave the original files as is. You should have 4 `.py` files now under `source/`.

Now both client and server must implement `MODE: 3`, which signifies the `authentication` handshake.

1. Upon successful `connect`, client must first send `3` (`int` converted to `bytes`) to the server, followed by two messages:

   - M1: The authentication message size in bytes
   - M2: The authentication message itself
   - Then the client <span style="color:#f77729;"><b>expects</b></span> to read <span style="color:#f77729;"><b>FOUR</b></span> messages from the server:
     - M1 from server: size of incoming M2 in bytes
     - M2 from server: signed authentication message
     - another M1 from server: size of incoming M2 in bytes (this is `server_signed.crt`)
     - another M2 from server: `server_signed.crt`

2. Upon receiving `MODE: 3`, the server will send these four messages above _immediately_, in sequence one after another. There's no need to send another `MODE` to the client because are still under the AP -- and the sequence <span style="color:#f77729;"><b>must be</b></span> as such: client send MODE 3, then <span style="color:#f77729;"><b>TWO</b></span> messages, and expecting <span style="color:#f77729;"><b>FOUR</b></span> messages from the server.

3. Once the client receives both replies from the server, it will perform the `CHECK` as specified above using the CA's public key extracted from `csertificate.crt`. If check is successful, the regular non-secure FTP should proceed where we can key in the file names one by one for the server to receive. Otherwise, send a `MODE 2` to close connection immediately.

You <span style="color:#f7007f;"><b>must</b></span> implement these `MODE` as specified above, and keep the original `MODE 0, 1, 2` as-is. The idea is to stick to this authentication and file transfer protocol so that _any_ client script can communicate with _any_ server script.
{:.error}

> Yes, we will test your client script against our server script, and your server script against our client script. It's gonna be fun!

You might want to refer to the [fifth page](https://natalieagus.github.io/50005/assignments/pa2_5_crypto) of this assignment to get started with some basic functionalities of Python `cryptography` module.

Finally, Here's a recap of the `MODE` for AP:

- `0`: client will send `M1`: size of filename, and `M2`: the filename (no need to modify, same as original)
- `1`: client will send `M1`: size of data block in `M2`, and `M2`: plain, unencrypted blocks of data to the server (no need to modify, same as original)
- `2`: client closes connection (no need to modify, same as original)
- `3`: client begins authentication protocol (NEW for Task 1)

### Grading

We will <span style="color:#f77729;"><b>manually</b></span> check the implementation of your `MODE 3` in both Client and Server scripts.

### Commit Task 1

Save your changes and commit the changes (assuming your current working directory is `source/`):

```
git add ServerWithSecurityAP.py ClientWithSecurityAP.py auth/server_signed.crt auth/server_private_key.pem
git commit -m "feat: Complete Task 1"
```
