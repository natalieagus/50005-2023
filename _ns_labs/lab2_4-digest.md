---
title: Signed Message Digests
permalink: /ns_labs/lab2_4-sign
key: ns-labs-lab2_4-sign
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

In class,  we also learned how a signed message digest could be used to guarantee the integrity of a message. Signing the <span style="color:#f77729;"><b>digest</b></span> instead of the message itself gives much better <span style="color:#f77729;"><b>efficiency</b></span>.

In the final task, we will create and sign a message digest, and <span style="color:#f77729;"><b>verify</b></span> it with the corresponding public key.

Open `3_sign_digest.py` and let's begin. 

## Generate RSA Key-Pair
### Task 3-(1,2) 
`TASK 3-(1,2):`{:.info} Before we can sign any digest, we need to first generate the asymmetric key pair. It is very simple to do so (you will also encounter this in Programming Assignment 2):
```
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=1024,
)

public_key = private_key.public_key()
```
The first argument dictates the value of `e` (the public exponent). Almost everyone uses `65537` (for security purposes) as recommended [here](https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/). The second argument dictates the key size (1024 bits in for this task).

##  Using the Keys
Afterwards, we can use the private or public key for encryption or decryption. However, there are some terminologies that you need to know.

### Encrypt and Decrypt
Encryption of a message with a <span style="color:#f77729;"><b>public key</b></span> is normally labeled as `encryption` in the API, and <span style="color:#f77729;"><b>decryption</b></span> with a private key is labeled as `decryption` (names that you will expect). Here's an example to get you started:
```python
data_bytes = b"Lorem ipsum dolor sit amet"

encrypted_data_bytes = public_key.encrypt(
    data_bytes,
    padding.OAEP(
        mgf=padding.MGF1(hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)

decrypted_data_bytes = private_key.decrypt( # 128 bytes long
    encrypted_data_bytes,
    padding.OAEP(
        mgf=padding.MGF1(hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)

assert decrypted_data_bytes == data_bytes

print("encrypted_data_bytes", encrypted_data_bytes) # ciphertext
print("decrypted_data_bytes", decrypted_data_bytes) # same as data_bytes
```

Note that for `encrypt` with `public_key`, the length of data bytes depends on the <span style="color:#f77729;"><b>padding</b></span> scheme. This example uses [`OAEP`](https://link.springer.com/referenceworkentry/10.1007/978-1-4419-5906-5_150) scheme, but there are other schemes too such as `PKCS1v15`, etc. You can read more about alternative paddings and their parameters [here](https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/#module-cryptography.hazmat.primitives.asymmetric.padding).

It is <span style="color:#f7007f;"><b>important</b></span> to know how many bytes the padding will add at *minimum*. 
- With 1024 RSA-key, the maximum length of message that it can encrypt is also 1024 bits (128 bytes)
- OAEP with `SHA-256` takes 66 bytes of padding overhead, leaving you with only 62 bytes of content to encrypt/decrypt at a time
- You will encounter this as well in Programming Assignment 2


### Sign and Verify
If you want to <span style="color:#f77729;"><b>encrypt</b></span> a message using the <span style="color:#f77729;"><b>private key</b></span>, the keyword you should look for in the API is `sign`, and the output is commonly called as a <span style="color:#f77729;"><b>signature</b></span>
* Conversely, if you want to <span style="color:#f77729;"><b>decrypt</b></span> the output signature, the keyword in the API is `verify`. There will be no output here if the verification succeeds (the decrypted signature matches the initial message), but a `VerficationError` will be raised if otherwise.  
* But don't be fooled! Verification *is a decryption*, and signing *is an encryption*. They just have different names that are tied to their *purpose*.

Here's an example to get you started:
```python
data_bytes = b"Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna" # 121 bytes

signature = private_key.sign(
    data_bytes,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH,
    ),
    hashes.SHA256(),  # Algorithm to hash the file_data before signing
)

try:
    public_key.verify(
        signature,
        data_bytes,
        padding.PSS( 
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH,
        ),
        hashes.SHA256(),
    )
    print("Verification Succeeds")
except:
    print("Verification Fails")
    exit()
```

Note that the length of `data_bytes` can be arbitrarily long because it will be <span style="color:#f77729;"><b>hashed</b></span> automatically (third argument of `sign`) before encrypted with the private key. The <span style="color:#f77729;"><b>output</b></span> length of `SHA-256` is 32 bytes only (256 bits) which when added with the padding, will still be less than 128 bytes. 

We also use a <span style="color:#f77729;"><b>different</b></span> padding scheme for the signature (`PSS`) and not `OAEP` because of the *nature* of the padding. You may read more about it [here](https://www.cryptosys.net/pki/manpki/pki_rsaschemes.html) or in any other online source materials but it is our of our syllabus.

## Creating a Digest
### Task 3-(3,4) 
`TASK 3-(3,4):`{:.info} In order to create a message digest, you must first create a <span style="color:#f77729;"><b>hash</b></span> instance, then use it with `update(input)` and `finalize()` methods.

```python
data_bytes = b"Lorem Ipsum" # 11 bytes

hash_function = hashes.Hash(hashes.SHA256())

hash_function.update(data_bytes)
message_digest_bytes = hash_function.finalize() # 32 bytes
print("message_digest_bytes", message_digest_bytes)
```

The print output is:
```
message_digest_bytes b'\x03\r\xc1\xf96\xc3AZ\xff?3W\x165\x15\x19\r4z(\xe7X\xe1\xf7\x17\xd1{\xaeE5A\xc9'
```

> Is the length of `message_digest_bytes` always the same (32 bytes), regardless of the length of the input `data_bytes`?

### Task 3-(5-8) 
`TASK 3-(5-8):`{:.info} With all the explanations above, you should be able to complete these tasks. Fill in your answer under the `TODO` for this task. 


## Summary
Finally, if you scroll below you will find these four function calls:
```python
if __name__ == "__main__":
    enc_digest("original_files/shorttext.txt")
    enc_digest("original_files/longtext.txt")
    sign_digest("original_files/shorttext.txt")
    sign_digest("original_files/longtext.txt")
```

Here we test encryption of digest using public key (`enc_digest`) and encryption of digest using private key (`sign_digest`), and we test each with two files of differing length. Study its output carefully and head to edimension to answer the rest of the questionnaires. 

