# John the Ripper Room

## My Personal Study Notes

These are my personal notes from the TryHackMe John the Ripper room. I wrote them to record what I personally understood while going through the room. They are not intended to reproduce the TryHackMe material word for word.

---

# Task 1: Introduction

## What is John the Ripper?

John the Ripper is a password-cracking tool. It is commonly used to crack password hashes and other password-protected data.

It can work with different types of password hashes and, in the Jumbo version, can also work with things such as Windows NTLM hashes, SSH private keys, and password-protected archives.

The main idea I took from this task is that John the Ripper is used to test password candidates against a stored password hash or a converted representation of password-protected data.

---

# Task 2: Basic Terms

## What is a hash?

A hash is a fixed-length value produced when data is processed through a hashing algorithm.

For example, a password can be passed through a hashing algorithm and the result is a hash value.

## What makes hashes different from encryption?

One important difference I learned is that hashing is designed to be a one-way process. Encryption is designed so that the original data can be recovered through decryption when the correct key is available.

A hash is therefore not normally "decrypted" to get the original password.

## How does John the Ripper crack hashes?

At first this was confusing to me because hashes are designed to be one-way.

John does not reverse the hashing algorithm and recover the original password directly.

Instead, John generates or obtains possible password candidates and hashes those candidates using the appropriate hashing algorithm. It then compares the resulting hash with the target hash.

If a generated candidate produces the same hash as the target, John has found the password candidate that corresponds to that hash.

One common approach is a dictionary or wordlist attack. John can take words from a wordlist and test them against the target hash. It can also modify words using rules to generate additional password candidates.

## John the Ripper Jumbo

The more advanced version I came across is called **John the Ripper Jumbo**.

The Jumbo version adds support for many additional hash and encrypted-file formats.

---

# Task 3: Installation and Wordlists

## Wordlists

A wordlist is a file containing many possible password candidates.

Wordlists are important because John can use them to test large numbers of possible passwords against a target hash.

One wordlist commonly used on Kali Linux is:

`rockyou.txt`

A common location for wordlists on Kali Linux is:

`/usr/share/wordlists`

## Basic command structure

The basic structure I learned is:

```text
john [options] [hash file]
```

I understand this as:

`john` starts the John the Ripper program.

`[options]` specify how I want John to operate.

`[hash file]` identifies the file containing the hash or password data that John should work on.

John can also automatically detect supported hash formats in many situations, so I do not always have to manually specify the format.

## Listing supported formats

The command:

```bash
john --list=formats
```

displays the hash formats supported by my John installation.

## `raw`

I learned that `raw` is used as part of format names for hashes that are treated as raw or unmodified hash values, for example raw SHA-1 or raw SHA-256.

## Task 3 questions

**QN1:** `rockyou.com`

**QN2:** `biscuit`

**QN3:** `sha1`

**QN4:** `kangaroo`

**QN5:** `sha256`

---

# Task 4: Cracking Basic Hashes

This task took me into actually cracking basic hashes with John the Ripper.

## Questions

**QN6:** `micro phone`

**QN7:** `whirlpool`

**QN8:** `colossal`

---

# Task 5: Cracking Windows Authentication Hashes

## Authentication hashes

Authentication hashes are hashes created from credentials such as passwords and stored so that a system can verify a password without storing the plaintext password itself.

A common example in Windows is **NTLM**.

## SAM

Windows stores local user account information in the **Security Account Manager (SAM)** database.

Access to the SAM is restricted and normally requires appropriate privileges.

There are tools and techniques that can extract password hashes from Windows systems. One example mentioned in the room was **Mimikatz**.

## Questions

**QN1:** `nt`

**QN2:** `mashroom`

---

# Task 6: Cracking `/etc/shadow` Hashes

On Linux, `/etc/shadow` normally contains password hashes and related password-account information.

Access to `/etc/shadow` is restricted. Normally, root privileges or equivalent permissions are required to read it.

## `/etc/passwd` and `/etc/shadow`

Linux systems separate general account information from protected password information.

`/etc/passwd` contains account information that is generally readable.

`/etc/shadow` contains the protected password hashes and related information.

## `unshadow`

John provides a utility called `unshadow`.

It combines the passwd and shadow files into a format that John can use.

The basic syntax is:

```bash
unshadow [passwd file] [shadow file]
```

For example:

```bash
unshadow /etc/passwd /etc/shadow > mypasswd
```

The resulting file can then be supplied to John.

I initially thought of `unshadow` as a tool specifically for cracking Linux hashes. More accurately, it prepares the passwd and shadow information so that John can process it.

## Question

**QN1:** `1234`

---

# Task 7: Single Crack Mode

Single Crack Mode is a John the Ripper mode that uses information associated with the account, such as the username and GECOS information, to generate password candidates.

It can apply word mangling techniques to that information.

## Word mangling

Word mangling means modifying a base word to generate possible password variations.

For example, if the base word is:

`Kiyimba`

possible variations could include:

```text
KIyimba
KIYimba
KIYImba
KIYIMba
Kiyimba1
Kiyimba2
Kiyimba3
Kiyimba4
```

The idea is that users often create passwords by making predictable changes to words they already know.

John can generate candidates from the available information instead of relying only on a normal dictionary.

## GECOS information

I also learned about **GECOS information**.

This can contain additional information associated with a Unix user account. John can use information from the account when operating in Single Crack Mode and generate possible password variations from it.

## Using Single Crack Mode

The option for Single Crack Mode is:

```bash
--single
```

For example:

```bash
john --single --format=raw-sha256 hash.txt
```

I understand `--single` as the option that tells John to use Single Crack Mode.

## Question

**QN1:** `Jok3r`

---

# Task 8: Custom Rules

Custom rules are one of the features I found particularly useful in John the Ripper.

They allow me to control how John modifies words when generating password candidates.

This is useful because users can be predictable when following password-complexity requirements.

For example, an organisation may require passwords to contain a capital letter, numbers, and symbols. Users may still follow predictable patterns such as:

```text
Password1!
Password2!
Password3!
```

The exact examples above are my own examples to explain the idea. They are not examples copied from the TryHackMe room.

## Where custom rules are defined

John's configuration file is commonly called:

```text
john.conf
```

On Unix-like systems, a system-wide configuration file can be located at:

```text
/etc/john/john.conf
```

The exact location can depend on the installation.

## Defining a custom rule

A custom rule is defined in a section such as:

```text
[List.Rules:MyRule]
```

The name after the colon is the name I give the rule.

The actual rule syntax then defines how John should modify the words.

## Rule syntax I learned

Some of the rule syntax I came across includes:

`Az`

This can append characters to the end of a word when used with the appropriate character specification.

`c`

This is used to capitalize characters according to John's rule syntax.

Character classes can be used to specify characters that John should use.

Examples include:

```text
[a-z]
```

Lowercase letters.

```text
[A-Z]
```

Uppercase letters.

```text
[A-z]
```

Both uppercase and lowercase letters.

```text
[0]
```

Only the character `0`.

```text
[0-9]
```

Numbers from 0 to 9.

```text
[a]
```

Only the character `a`.

```text
[fgtfr]
```

Characters from the specified set.

The exact behavior of a rule depends on how the rule syntax is constructed.

## Using custom rules

The general idea is to select the wordlist, specify the rule set, and provide the hash file.

For example:

```bash
john --wordlist=[path to wordlist] --rules=[rule name] [hash file]
```

One of the questions in the room used:

```text
--rules=THMRules
```

## Questions

**QN1:** `password complexity predictability`

**QN2:** `A-z"[A-Z]"`

**QN3:** `--rules=THMRules`

---

# Task 9: Cracking Password-Protected ZIP Files

I learned that John can be used to crack passwords protecting ZIP archives.

John does not simply take the ZIP archive as an ordinary hash file. A helper utility is used first to extract information from the ZIP file into a format John can work with.

The tool is:

```text
zip2john
```

A basic example is:

```bash
zip2john file.zip > ziphash.txt
```

The resulting file can then be supplied to John.

## About `unzip`

I noticed that the command:

```bash
unzip [file name]
```

appeared in the room.

I initially found this confusing because it is different from the John-specific commands. The important distinction is that `unzip` is a normal Linux utility for extracting ZIP archives, while `zip2john` is used to prepare information from a password-protected ZIP archive for John.

## Questions

**QN1:** `Pass123`

**QN2:** `THM{w3ll_d0n3_h4sh_r0y4l}`

---

# Task 10: Cracking Password-Protected RAR Archives

This process is similar to cracking password-protected ZIP archives.

Instead of `zip2john`, I use:

```text
rar2john
```

A basic example is:

```bash
rar2john file.rar > rarhash.txt
```

The resulting file can then be supplied to John.

## Questions

**QN1:** `password`

**QN2:** `THM{r4r_4rch1ve5_th15_t1m3}`

---

# Task 11: Cracking SSH Keys with John

SSH private keys can also be protected with a passphrase.

If I obtain an encrypted SSH private key, I can use a John helper utility to convert the key into a format that John can process.

The relevant utility is:

```text
ssh2john
```

A typical workflow is:

```bash
ssh2john id_rsa > sshhash.txt
john sshhash.txt
```

The idea is that John is not directly reversing the SSH key. The helper utility extracts the relevant password-verification information from the private key, and John then attempts to recover the passphrase.

Once the passphrase is recovered, it can be used to access the protected private key.

## Question

**QN1:** `mango`

---

# Task 12: Further Reading

This was the final part of the room.

The room provided further reading for learning more about John the Ripper and its capabilities.

---

# Commands I Want to Remember

## Basic John syntax

```bash
john [options] [hash file]
```

## List supported formats

```bash
john --list=formats
```

## Single Crack Mode

```bash
john --single --format=raw-sha256 hash.txt
```

## Combine Linux passwd and shadow files

```bash
unshadow /etc/passwd /etc/shadow > mypasswd
```

## Convert a ZIP archive for John

```bash
zip2john file.zip > ziphash.txt
```

## Convert a RAR archive for John

```bash
rar2john file.rar > rarhash.txt
```

## Convert an SSH private key for John

```bash
ssh2john id_rsa > sshhash.txt
```

## Use a custom rule

```bash
john --wordlist=[path to wordlist] --rules=[rule name] [hash file]
```

---

# My Main Understanding

The main thing I learned from this room is that John the Ripper is not simply a tool that "decrypts" hashes.

Instead, I give John a target hash or a file containing password-related data, and John generates password candidates using different techniques. It hashes or tests those candidates using the appropriate format and compares the results with the target.

The different modes give me different ways of generating candidates.

A wordlist attack uses a list of possible passwords.

Single Crack Mode uses information associated with a user account and modifies it to generate likely passwords.

Custom rules allow me to define predictable patterns that John should use when modifying words.

John can also work with password-protected files such as ZIP and RAR archives and encrypted SSH private keys after the relevant `*2john` utility converts them into a format John understands.

The most important command structure I want to remember is:

```text
john [options] [hash file]
```

The `john` part starts the program, the options control what I want John to do, and the final argument identifies the data John should work on.

These notes represent my own understanding after completing the room. They are meant for future revision rather than as a copy of the TryHackMe material.
