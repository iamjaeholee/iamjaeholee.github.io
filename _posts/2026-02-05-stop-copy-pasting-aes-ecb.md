---
layout: post
title: "Beyond AES-256: Understanding Block Cipher Modes of Operation"
date: 2026-02-05 16:30:00 +0900
categories: [Security, Engineering]
tags: [AES, Cryptography, Java, Backend]
author: Boksepyeonsal
---

"Please encrypt the DB using AES-256."

When we receive this instruction as junior developers, we usually Google it and copy-paste the code. However, you might notice that the string passed into `Cipher.getInstance()` varies slightly:

* `AES/ECB/PKCS5Padding` (?)
* `AES/CBC/PKCS5Padding` (?)
* `AES/GCM/NoPadding` (?)

"Isn't AES just AES? What are all these suffixes?"

If you ignore these details and choose something like **ECB**, you might face a catastrophe where data patterns remain visible even after encryption. Today, we will dive into the heart of cryptography, **'Block Cipher Modes of Operation'**, using the language of developers: **Code**.

---

## 1. Fundamentals: Why XOR?

Every explanation of encryption modes involves the `XOR` operation. Why XOR, when we have addition (+) and multiplication (*)? There are three decisive reasons.

1.  **Perfect Reversibility**: If you encrypt `A` by XORing it with a `Key`, XORing the result with the `Key` again perfectly restores `A`.
    * `A ⊕ Key = Cipher`
    * `Cipher ⊕ Key = A`
2.  **50:50 Diffusing**: When any data is XORed with a random key, the distribution of 0s and 1s in the result becomes 50/50. It is the **most efficient way to erase data traces**.
3.  **Overwhelming Speed**: It is the cheapest and fastest operation for the CPU.

The operating modes below are divided by **"when and with what"** this XOR is mixed.

---

## 2. The Worst Choice: ECB (Electronic Codebook)

This is the simplest "Cookie Cutter" approach. Data is cut into blocks (16 bytes) and encrypted independently in a loop.



### 💻 Code Logic (Pseudo-code)

```python
def encrypt_ecb(plain_blocks, key):
    cipher_text = ""
    # Each block is a stranger to the others (Independent)
    for block in plain_blocks:
        cipher_text += AES_encrypt(block, key) 
    return cipher_text

```

### ❌ Fatal Flaw: Pattern Exposure

If the input is the same, the output is unconditionally the same. When encrypting images, this creates a **critical vulnerability where the shape remains visible**.

Seeing is believing. The famous "ECB Penguin" visualization demonstrates why this is dangerous.

> **Practical Rule 1**: `AES/ECB` is a security vulnerability. Never use it.

---

## 3. Chaining Blocks: CBC (Cipher Block Chaining)

To solve the ECB problem, this method **"mixes (XOR) the result of the previous block into the next block."** It has been the standard for a long time.

### 💻 Code Logic (Pseudo-code)

```python
def encrypt_cbc(plain_blocks, key, iv):
    cipher_text = ""
    prev_cipher = iv  # Starts with an Initialization Vector (IV)
    
    for block in plain_blocks:
        # 1. Mix with 'previous ciphertext' before encryption (XOR)
        mixed_block = xor(block, prev_cipher)
        
        # 2. Encrypt the mixed block
        current_cipher = AES_encrypt(mixed_block, key)
        
        cipher_text += current_cipher
        prev_cipher = current_cipher  # Current result becomes the ingredient for the next
    return cipher_text

```

### ✅ Features

* **Pros**: Even with the same "Hello", the result changes completely depending on the previous content. (Pattern Destruction)
* **Cons**: Since `prev_cipher` is required to run the next loop, **parallel processing** is impossible, making it slower.

---

## 4. The "Stream" Approach: CFB, OFB, CTR

This is where it gets interesting. These three modes use a **Block Cipher (AES)**, but they behave like a **Stream Cipher**. The reason is that they **"do not encrypt the data directly."**

### 🌊 Common Principle: "Key Stream"

1. The encryption engine (AES) runs on its own, ignoring the plaintext, to continuously generate a **'Disposable Key Stream'**.
2. The data (plaintext) is simply **XORed** with this generated key.

### 🚀 Benefits

1. **No Padding Needed**: Since you only need to XOR, you don't need to force the data length to 16 bytes.
2. **Real-time**: Even if only 1 bit or 1 byte comes in, it can be encrypted and sent immediately. (Good for video calls, chatting).

---

## 5. Code Comparison of the Trio

### ① CFB (Cipher Feedback): Tail-Biting

The **'ciphertext'** from the previous step is put back into the encryption engine to create the key.

```python
def encrypt_cfb(plain_blocks, key, iv):
    feedback = iv
    for block in plain_blocks:
        # 1. Encrypt 'previous ciphertext (feedback)' to make a key
        key_stream = AES_encrypt(feedback, key)
        
        # 2. XOR data with the key
        cipher_block = xor(block, key_stream)
        
        # 3. Pass ciphertext to the next step (Error Propagation occurs!)
        feedback = cipher_block 

```

* **Feature**: If a bit error occurs during transmission, the key generation material (`feedback`) for the next step is corrupted, so **the error propagates to the next block**.

### ② OFB (Output Feedback): The Lonely Engine

The **'output'** of the encryption engine is put back as input. Plaintext/Ciphertext is not involved in key generation.

```python
def encrypt_ofb(plain_blocks, key, iv):
    output = iv
    for block in plain_blocks:
        # 1. Encrypt the engine output again to make a key
        output = AES_encrypt(output, key)
        key_stream = output
        
        # 2. XOR data with the key
        # Even if plaintext is corrupted, key generation (output) is unaffected 
        # -> No Error Propagation
        cipher_block = xor(block, key_stream)

```

* **Feature**: Even if a transmission error occurs, only that specific bit is corrupted. The rest remains intact. (Good for video streaming).

### ③ CTR (Counter): The King of Speed (Modern Standard)

Regardless of the data, it encrypts a **'Counter (Number)'** that increases by 1 to create the key.

```python
def encrypt_ctr(plain_blocks, key, nonce):
    # Loop order doesn't matter! (Parallel processing possible)
    for i, block in enumerate(plain_blocks):
        counter = nonce + i
        
        # 1. Encrypt the number to make a key
        key_stream = AES_encrypt(counter, key)
        
        # 2. XOR data with the key
        cipher_block = xor(block, key_stream)

```

* **Feature**: Each block is **Independent**. Multi-core CPUs can encrypt simultaneously, making it **extremely fast**.

---

## 6. Conclusion: The Developer's Guide

| Situation | Recommended Mode | Reason |
| --- | --- | --- |
| **Modern Standard (Best)** | **GCM** | Based on **CTR mode**, so it's fast (Parallel) and even provides data integrity checks (Authentication). |
| **Legacy Compatibility** | **CBC** | Use if you must communicate with older systems. (Slow). |
| **Forbidden** | **ECB** | No security. Use only for testing. |

Now, when you see `AES/GCM/NoPadding` in Java or Python, you can understand: **"Ah, it makes a key stream with a counter and XORs it, so no padding is needed and it's fast!"**