# Public Key Cryptography Basics

**Write-up by: Nicholas Kiyimba**

## Introduction

This section introduced me to public key cryptography and the areas it covers. From my understanding, public key cryptography helps provide:

- Confidentiality
- Integrity
- Authenticity
- Authentication

The section then introduced asymmetric encryption, RSA, Diffie-Hellman key exchange, Secure Shell (SSH), digital signatures and certificates, PGP/GPG, and some common cryptographic attacks and concepts.

---

## Task 2 — Common Usage of Asymmetric Encryption

One of the main things I learned was the common usage of asymmetric encryption.

Asymmetric encryption is slower compared to symmetric encryption. Because of this, asymmetric cryptography can be used to securely exchange a key that will then be used with symmetric encryption.

For example, if Person A wants to communicate with Person B, asymmetric cryptography can be used to securely establish or exchange a symmetric key. After that, symmetric encryption can be used for the actual communication because it is much faster and more efficient.

My understanding is therefore:

**Asymmetric cryptography → securely establish/exchange the symmetric key → symmetric cryptography → fast communication.**

---

## Task 3 — RSA

RSA is an encryption algorithm based on public and private keys.

From the mathematics I learned, RSA involves values such as:

- `p`
- `q`
- `n`
- `e`
- `d`
- `m`

`p` and `q` are prime numbers selected during RSA key generation.

The value of `n` is calculated by multiplying the two primes:

```text
n = p × q
```

The RSA public key is:

```text
(n, e)
```

and the private key is:

```text
(n, d)
```

The security of RSA is related to the difficulty of factoring the large composite number `n` into its original prime factors `p` and `q`.

### My RSA calculation attempt

I initially tried to use very small values to make the mathematics easier.

I first considered:

```text
p = 2
q = 4
```

but then noticed that `4` is not a prime number, so I changed it to:

```text
p = 2
q = 3
```

The value of `n` should therefore be:

```text
n = p × q
n = 2 × 3
n = 6
```

Euler's totient can be calculated as:

```text
φ(n) = (p - 1)(q - 1)
```

or equivalently:

```text
φ(n) = n - p - q + 1
```

for two distinct primes.

I also learned that `e` and `d` must satisfy:

```text
e × d ≡ 1 (mod φ(n))
```

This means that the product of `e` and `d` must leave a remainder of `1` when divided by `φ(n)`.

My original attempt at choosing `e = 7` and `d = 13` was incorrect for the small example I was using. This was one of the calculations I needed to correct while reviewing my understanding.

The general idea I understood from RSA is that Person A can encrypt a message using the recipient's public key, and the recipient can use the corresponding private key to recover the plaintext.

---

## Task 4 — Diffie-Hellman Key Exchange

Diffie-Hellman was one of the areas where I spent the most time trying to understand the mathematics.

I did not want to simply reproduce the example from TryHackMe, so I created my own example using different letters and values.

### My variable system

Instead of using the exact letters from the example, I used my own notation:

- `M` = the large prime number
- `R` = the generator
- `L` = Alice's private integer
- `F` = Bob's private integer
- `A` = Alice's public key
- `B` = Bob's public key

The public values are agreed upon by Alice and Bob.

The private integers are chosen individually and must not be shared.

### Public values

For my example I used:

```text
M = 17
R = 23
```

Alice chose:

```text
L = 2
```

Bob chose:

```text
F = 10
```

These private integers represent the private values that Alice and Bob keep secret.

### Calculating Alice's public value

Alice calculates her public value using:

```text
A = R^L mod M
```

Therefore:

```text
A = 23^2 mod 17
```

Since:

```text
23 mod 17 = 6
```

we get:

```text
A = 6^2 mod 17
A = 36 mod 17
A = 2
```

Therefore Alice's public value is:

```text
A = 2
```

### Calculating Bob's public value

Bob calculates his public value using:

```text
B = R^F mod M
```

Therefore:

```text
B = 23^10 mod 17
```

This calculation gives:

```text
B = 4
```

Therefore Bob's public value is:

```text
B = 4
```

The public values can be exchanged publicly. An eavesdropper can see the public values, but the private integers remain secret.

### Calculating the shared secret

After exchanging their public values, Alice and Bob can independently calculate the same shared secret.

Alice calculates:

```text
K = B^L mod M
```

Bob calculates:

```text
K = A^F mod M
```

Using the values above:

```text
K = 4^2 mod 17
K = 16
```

Bob calculates:

```text
K = 2^10 mod 17
K = 16
```

Therefore both sides obtain:

```text
K = 16
```

This demonstrates the main idea I was trying to understand: Alice and Bob can arrive at the same shared secret without directly sending that secret across the network.

The shared secret is not simply a password. It is shared secret material that can be used to establish symmetric encryption keys for protected communication.

### Understanding the generator

One question I had while studying Diffie-Hellman was what the generator `G` actually means.

In the Diffie-Hellman context, the generator is a carefully selected public value used as the base for the modular exponentiation. It is not a "generator" in the ordinary sense of a tool that generates something. It is a mathematical generator of a suitable cyclic subgroup.

---

## TryHackMe Diffie-Hellman Questions

The task also included four questions.

The values given were:

```text
P = 29
G = 5
a = 12
b = 17
```

I approached the questions by reconstructing the complete Diffie-Hellman process instead of calculating each answer independently.

### Phase 1 — Choose the public variables

Alice and Bob agree on:

```text
P = 29
G = 5
```

`P` is the prime modulus and `G` is the generator.

### Phase 2 — Choose the private integers

Alice chooses:

```text
a = 12
```

Bob chooses:

```text
b = 17
```

These private values are not shared.

### Phase 3 — Calculate the public keys

Alice calculates her public key:

```text
A = G^a mod P
```

Therefore:

```text
A = 5^12 mod 29
A = 7
```

So Alice's public key is:

```text
A = 7
```

Bob calculates his public key:

```text
B = G^b mod P
```

Therefore:

```text
B = 5^17 mod 29
B = 9
```

So Bob's public key is:

```text
B = 9
```

Alice and Bob can then exchange these public keys publicly.

### Phase 4 — Calculate the shared key

Alice uses Bob's public key and her private value:

```text
K = B^a mod P
```

Therefore:

```text
K = 9^12 mod 29
K = 24
```

Bob uses Alice's public key and his private value:

```text
K = A^b mod P
```

Therefore:

```text
K = 7^17 mod 29
K = 24
```

Both calculations produce:

```text
K = 24
```

This confirmed that my calculations were correct.

### Answers

**Question 1:**

Given:

```text
P = 29
G = 5
a = 12
```

Find `A`:

```text
A = 5^12 mod 29
A = 7
```

**Answer: `7`**

**Question 2:**

Given:

```text
P = 29
G = 5
b = 17
```

Find `B`:

```text
B = 5^17 mod 29
B = 9
```

**Answer: `9`**

**Question 3:**

Given:

```text
P = 29
a = 12
B = 9
```

Bob's shared key calculation:

```text
K = B^a mod P
K = 9^12 mod 29
K = 24
```

**Answer: `24`**

**Question 4:**

Given:

```text
P = 29
b = 17
A = 7
```

Alice's shared key calculation:

```text
K = A^b mod P
K = 7^17 mod 29
K = 24
```

**Answer: `24`**

---

## Task 5 — Secure Shell (SSH)

Secure Shell, commonly known as SSH, is used to log into a remote machine over an encrypted connection.

SSH uses cryptographic technologies to protect the connection.

One of the things I learned was that during the connection process, the server has a host public key that the client can use to verify that it is communicating with the correct server. This helps protect the user against an attacker interfering with the connection.

I also learned that SSH can be used to generate public and private key pairs.

The command I learned for generating SSH keys was:

```bash
ssh-keygen
```

I also learned that the manual can be accessed using:

```bash
man ssh-keygen
```

The `-t` option can be used to specify the key type. For example:

```bash
ssh-keygen -t rsa
```

This generates an RSA key pair.

The generated key pair contains:

- A public key
- A private key

The public key can be shared, while the private key must remain secret and protected on the machine that owns it.

A private key should never be unnecessarily shared because anyone who obtains a usable private key may be able to authenticate as its owner, depending on the configuration.

### SSH keys in CTFs

During penetration-testing CTFs, private SSH keys can sometimes be found on compromised machines. If an attacker obtains a private key and the corresponding account allows SSH authentication with that key, the key can potentially be used as a way to access the machine again.

This is a CTF/attack scenario and is not a recommendation to leave private keys behind on real systems.

The task also involved locating an SSH private key file and inspecting its contents to identify the algorithm.

---

## Task 6 — Digital Signatures and Certificates

### Digital signatures

A digital signature is used to provide authenticity and integrity for data.

My initial understanding was that a hash of a document or message could be protected using the sender's private key, and that the receiver could use the sender's public key to verify the signature.

The basic process I learned is:

1. A message or document is hashed.
2. A digital signature is generated using the sender's private signing key.
3. The message and signature are sent to the receiver.
4. The receiver uses the sender's public key to verify the signature.
5. The receiver can also hash the received data and use the verification process to confirm that the data corresponds to the signature.

If verification succeeds, this provides evidence that the data came from the holder of the corresponding private signing key and that the signed data has not been modified.

Digital signatures are therefore concerned with authenticity and integrity rather than confidentiality.

### Certificates

Certificates help prove the authenticity of a website.

Certificates are issued through certificate authorities (CAs).

I learned about a chain of trust involving:

- Root certificate authorities
- Certificate authorities/intermediate authorities
- Website certificates

A web browser can use the certificate presented by a website and verify its chain against trusted certificate authorities.

Certificates also have expiry dates, so they are not valid indefinitely.

---

## Task 7 — PGP and GPG

PGP/GPG is used to encrypt and sign information or data. It can also be used to decrypt protected information.

GPG is an open-source tool and supports a variety of Linux distributions.

I learned that it can be used for encryption, decryption, and key management.

One of the commands I learned is:

```bash
gpg
```

The `-e` option is used for encryption:

```bash
gpg -e
```

The `-c` option is used for symmetric encryption:

```bash
gpg -c
```

I also learned about importing a key.

For example:

```bash
gpg --import keyfile
```

The key file must first be imported into the GPG keyring before it can be used when the required private key is not already available.

After importing the required key, encrypted information can be decrypted using:

```bash
gpg --decrypt filename
```

### The GPG workflow I encountered

During the task, I initially attempted to decrypt the protected message before importing the required GPG private key.

The decryption failed because the required secret key was not available in the keyring.

The correct sequence was:

1. List the contents of the Task-7 directory.
2. Locate the required GPG private key.
3. Import the private key into the GPG keyring.
4. Decrypt the protected message.

After importing the private key, I successfully decrypted the protected message and retrieved the secret word.

This taught me that the required secret key must be available in the local GPG keyring before GPG can use it to decrypt the protected data.

I also learned that GPG can be useful for protecting sensitive information, although a purpose-built password manager is more appropriate for managing passwords than creating an ad-hoc password storage system.

---

## Conclusion

This section introduced me to several important cryptography concepts and tools.

The new terms I learned included:

### Cryptography

Cryptography refers to techniques used to protect information using cryptographic algorithms and ciphers.

### Dictionary attack

A dictionary attack involves trying words or commonly used password values from a prepared list in an attempt to discover a password or recover protected information.

### Cryptanalysis

Cryptanalysis is the study of methods used to analyze or break cryptographic systems and recover protected information without necessarily having the decryption key.

### Brute-force attack

A brute-force attack involves trying possible password or key combinations until the correct one is found.

### What I learned

Through this section, I learned about:

- Asymmetric encryption
- Symmetric encryption
- RSA
- Diffie-Hellman key exchange
- Secure Shell (SSH)
- Digital signatures
- Digital certificates
- PGP/GPG
- Brute-force attacks
- Dictionary attacks
- Cryptography
- Cryptanalysis

The biggest practical lessons for me were understanding why asymmetric and symmetric cryptography can be used together, working through the mathematics of RSA and Diffie-Hellman, understanding the role of SSH keys, learning how digital signatures and certificates help establish trust and integrity, and successfully managing a GPG private key before decrypting protected data.

---

## TryHackMe Mission Debrief

### Win — GPG

I successfully imported the GPG private key into the keyring and then decrypted the protected message to retrieve the secret word.

Flow:

- Listed Task-7 directory contents
- Imported GPG private key
- Decrypted message successfully

### Win — SSH

I successfully navigated to Task-5, located the SSH private key file, and extracted its contents to identify the algorithm.

Flow:

- Navigated to Task-5 directory
- Listed files to locate SSH key
- Inspected private key contents

### Improvement — GPG

I initially attempted decryption without first importing the required GPG private key.

GPG returned error code 2 because no secret key was available.

The correct sequence was to import the key first and then perform the decryption.

Flow:

- Attempted `gpg --decrypt` without the required key
- Key import was initially omitted
- Decryption failed with error code 2

### Action Points

- Master the GPG keyring workflow.
- Use `gpg --list-secret-keys` after importing a key to verify that the required secret key is present before attempting decryption.
- Learn `ssh-keygen` algorithm inspection.
- Use `ssh-keygen -lf <keyfile>` to extract the algorithm and fingerprint without exposing the entire private key in terminal scrollback.
