---
title: Symmetric Key Encryption on Image File
permalink: /ns_labs/lab2_3-sym-img
key: ns-labs-lab2_2-sym-img
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

In this task, we are going to encrypt an <span style="color:#f77729;"><b>image</b></span> with another symmetric key cryptography called the 3-DES. The 3-DES applies the DES cipher algorithm three times to <span style="color:#f77729;"><b>each</b></span> data block.

Data Encryption Standard (DES) <span style="color:#f77729;"><b>was</b></span> a US encryption standard for encrypting electronic data. It makes use of a 56-bit key to encrypt 64-bit blocks of plaintext input through repeated rounds of processing using a specialized function. Note that the DES 56-bit key is <span style="color:#f7007f;"><b>no longer considered adequate</b></span> in the face of modern computing. 
> In 2016, a major security vunerability in DES and 3DES was [revealed](https://nvd.nist.gov/vuln/detail/CVE-2016-2183). As a result, NIST (National Institute of Standards and Technology) <span style="color:#f7007f;"><b>depcrecated</b></span> DES and 3DES for <span style="color:#f7007f;"><b>all</b></span> applications by 2023. It has been <span style="color:#f7007f;"><b>replaced</b></span> by AES which is more secure and more robust. 


Nevertheless, we are going to experiment using 3-DES for this task and observe how image encryption brings about different issues. 

Open `2_encrypt_image.py` and let's begin. 

## Overview
There are 5 helper functions in the file, and these are mainly used to read columns of an image as bytes. You may leave these untouched:

```python
image_to_cols(image)
cols_to_image(cols)
col_to_bytes(col, top_down=False)
tuple_to_bytes(x)
bytes_to_col(x, length, top_down=False)
```

The main bulk of your task involves filling up the `TODO` in `enc_img` function. If you scroll down, you will see it being called with various images as input. 

### Loading the Image
The image is loaded in `enc_img` as such:

```python
# Load the image
im = Image.open(input_filename)
``` 

We will <span style="color:#f77729;"><b>encrypt</b></span> this image <span style="color:#f77729;"><b>column by column</b></span>.
> Yes we can encrypt the whole image as a flattened array, or row by row, but for the sake of this lab we will do it column by column so that we can observe a special output.

### Loading Pixel Values per Column
However, we can't use it directly for our encryption. We need to first convert it to an array of <span style="color:#f77729;"><b>columns</b></span>. As such, this helper function `image_to_cols` is used. This is not a Python class, but here's what `zip` and `*` does:

```python
# arr is a 3 by 2 by 3 matrix (3 rows, 2 columns)
arr = [
         [(1, 2, 3), (4, 5, 6)],
         [(7, 8, 9), (10, 11, 12)],
         [(13, 14, 15), (16, 17, 18)],
      ]
cols = list(zip(*arr)) 
print("cols", cols)
```

The above has the output:
```python
Note: cols is a 2 by 3 by 3 matrix (each "row" here is a column value of arr)

cols = [
         ((1, 2, 3), (7, 8, 9), (13, 14, 15)),
         ((4, 5, 6), (10, 11, 12), (16, 17, 18)),
       ]
```

### Convert Columns to Bytes 
The columns in the image consists of pixel values (tuples of 3 integers, representing RGB values), and we can't encrypt it straight away without converting them to bytes. The helper function `col_to_bytes` and `tuple_to_bytes` already did this for you:

```python
def col_to_bytes(col, top_down=False):
    """
    Helper function to convert each pixel tuple to bytes. Iterate the column top-down or bottom-up
    """
    out = []
    for pixel in col[:: (1, -1)[top_down]]:
        out.insert(0, tuple_to_bytes(pixel))
    return b"".join(out)


def tuple_to_bytes(x):
    """
    Helper function to convert a tuple of integers into bytes
    """
    return int.from_bytes(list(x), byteorder="big").to_bytes(
        3, byteorder="big"
    )
```

In short, if a `pixel` in `col` has the value of `(255, 255, 255)` (RGB), it will be converted into 3 bytes: `b'\xff\xff\xff'`. All pixels in the column will be converted into bytes and joined together, for instance:
```python
cols = [
         ((1, 2, 3), (7, 8, 9), (13, 14, 15)),
         ((4, 5, 6), (10, 11, 12), (16, 17, 18)),
       ]

for col in cols:
    column_bytes = col_to_bytes(col, True)
    print("column_bytes", column_bytes)
```

Has the output:
```python
column_bytes b'\x01\x02\x03\x07\x08\t\r\x0e\x0f'
column_bytes b'\x04\x05\x06\n\x0b\x0c\x10\x11\x12'
```

### Bottom-up vs Top-down
We can load the column bytes bottom-up or top-down. This will <span style="color:#f77729;"><b>affect</b></span> our encryption result later on. Consider our example above with image matrix:
```python
arr = [
         [(1, 2, 3), (4, 5, 6)],
         [(7, 8, 9), (10, 11, 12)],
         [(13, 14, 15), (16, 17, 18)],
      ]
``` 
The output above loads the column bytes "top-down", so we have column bytes:
```python
column_bytes b'\x01\x02\x03\x07\x08\t\r\x0e\x0f' # (1,2,3), (7,8,9), (13,14,15) in bytes
column_bytes b'\x04\x05\x06\n\x0b\x0c\x10\x11\x12' # (4,5,6), (10,11,12), (16,17,18) in bytes
```

If we load the column bytes "bottom-up", we will end up with column bytes:
```python
column_bytes b'\r\x0e\x0f\x07\x08\t\x01\x02\x03' # (13,14,15), (7,8,9), (1,2,3) in bytes
column_bytes b'\x10\x11\x12\n\x0b\x0c\x04\x05\x06' # (16,17,18), (10,11,12), (4,5,6) in bytes
```

> Notice how the order *within the tuple* must be preserved, just the way we pack each tuple is different. Don't worry, we have already done these mental gymnastics for you.

## Key Generation
The key in this case is already given to you:
```python
    # Key for 3DES
    key = b"\xb6\x11\xd5\xd7\x83\xb2,m"
```

## Create a Cipher 
### Task 2-1 
`TASK 2-1:`{:.info} You can create a Cipher with `TripleDES` using the `key` above as such:
```
cipher = Cipher(algorithms.TripleDES(key), mode)
```

The constructor for `Cipher` takes in two arguments:
* The first argument specifies the algorithm to be used and the <span style="color:#f77729;"><b>key</b></span>
* The second argument, <span style="color:#f77729;"><b>mode</b></span>, specifies how multiple blocks of data are handled by the encryption algorithm. It can be either <span style="color:#f77729;"><b>ECB</b></span> or <span style="color:#f77729;"><b>CBC</b></span> depending on whether the input argument `cbc` to `enc_image` is `True` or `False`. 

### ECB Mode
ECB stands for ‘electronic codebook’. When using ECB mode, <span style="color:#f77729;"><b>identical</b></span> input blocks always encrypt to the <span style="color:#f77729;"><b>same</b></span> output block.

### CBC Mode
CBC stands for 'cipher block chaining'. In CBC mode, the current block is added to the previous ciphertext block, and then the result is encrypted with the key (thus the word chained). Decryption is thus the reverse process, which involves decrypting the current ciphertext and then adding the previous ciphertext block to the result.

## Create a CipherContext instance for encryption
### Task 2-2 
`TASK 2-2:`{:.info} Afterwards, create a CipherContext instance from the `encryptor()` method,
```python
encryptor = cipher.encryptor()
```

It can be used to encrypt data as such:
```python
raw_bytes = b"abcdefghijklmnop"  # 16 bytes
print("raw_bytes", raw_bytes)
encrypted_bytes = (
    encryptor.update(raw_bytes) + encryptor.finalize()
)  # 16 bytes
print("encrypted_bytes", encrypted_bytes)
```

The output is:
```python
raw_bytes b'abcdefghijklmnop'
encrypted_bytes b'\x12\x96]\x00\x12\x0e\x80C*\x1b\xc7d\xd2\x10=\xeb'
```

> Note that if `ECB` mode is used, then the encrypted bytes above will <span style="color:#f77729;"><b>always</b></span> be the same. 

However, if we change the length of `raw_bytes` into 17 bytes, eg: `b"abcdefghijklmnopq"`, we will be faced with `ValueError: The length of the provided data is not a multiple of the block length.`. What happened?

## Pad Column Bytes
### Task 2-(3,4)
`TASK 2-(3,4):`{:.info} If you attempt to encrypt your data right away with this, it will fail because 3DES encrypts <span style="color:#f77729;"><b>64 bit blocks</b></span>. It expects the length of the data to be encrypted to be a multiple of 64. 

Hence we need to pad it. At first you need to create a padding instance, and use it as such:
```python
padder = padding.PKCS7(64).padder() 
test_bytes = b"Lorem" # 5 bytes
padded_test_bytes = padder.update(test_bytes) + padder.finalize() # 8 bytes
print("padded_test_bytes", padded_test_bytes)
```

The output is:
```python
padded_test_bytes b'Lorem\x03\x03\x03'
```

> PKCS7 padding is a <span style="color:#f77729;"><b>generalization</b></span> of PKCS5 padding (also known as standard padding). PKCS7 padding works by appending N bytes with the value of `chr(N)` (Python built-in function `chr`), where N is the number of bytes required to make the final block of data the same size as the block size.

With this, you should be able to complete Task 2-3 and 2-4. 

### Task 2-5 
`TASK 2-5:`{:.info} Encrypt the padded column bytes. Fill in your answer under the `TODO` for this task. 
> You should have enough information by now to complete this task.


## Observe the Output Images
When you have successfully completed Task 2-1 to 2-5, there will be 8 output images at `output/`. <span style="color:#f77729;"><b>Observe them</b></span> and think about the answers to these questions.

Look at all `ECB` outputs. 
1. Are there any differences between `top-down` and `bottom-up` output?
2. What kind of <span style="color:#f77729;"><b>security vulnerabilities</b></span> the `ECB` mode has? Will you use this to encrypt your images in real life? 

Then look at all `CBC` outputs:
1. Compare the result of `CBC` mode with `ECB` mode, what differences do you see? Is it an *improvement* (more secure)? 
2. Why do you think the outputs have the "smearing" effect? 
3. Does it still have some kind of security vulnerabilities? 
4. Do we want to prioritise `top-down` or `bottom-up` encryption, or will it depend on the image?



