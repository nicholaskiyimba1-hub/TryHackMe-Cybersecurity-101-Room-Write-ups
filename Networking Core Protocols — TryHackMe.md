# Networking Core Protocols — TryHackMe

## Overview

This room focuses on the **core protocols that operate at the application layer of computer networks**.

The goal was not simply to memorize protocol names and port numbers, but to understand what happens behind applications such as web browsers and email clients.

The protocols covered in this room were:

- DNS — Domain Name System
- WHOIS
- HTTP — Hypertext Transfer Protocol
- HTTPS — Hypertext Transfer Protocol Secure
- FTP — File Transfer Protocol
- SMTP — Simple Mail Transfer Protocol
- POP3 — Post Office Protocol version 3
- IMAP — Internet Message Access Protocol

A major takeaway from this room is that applications such as browsers and email clients are interfaces that communicate with servers using underlying network protocols.

---

# Task 1 — Introduction

This room is part of the TryHackMe networking learning path:

```text
Networking Concepts
        ↓
Networking Essentials
        ↓
Networking Core Protocols
        ↓
Networking Secure Protocols
```

The room assumes knowledge of:

- The OSI model
- The TCP/IP model
- Ethernet
- IP addressing
- Basic networking concepts

The practical exercises were performed using the TryHackMe AttackBox and provided lab machines.

---

# Task 2 — DNS: Remembering Addresses

## What is DNS?

**DNS stands for Domain Name System.**

Humans generally prefer names such as:

```text
example.com
google.com
github.com
```

Computers communicate using IP addresses such as:

```text
93.184.215.14
```

DNS provides the system that translates domain names into IP addresses.

A simple way to think about it is:

```text
Domain name
     ↓
    DNS
     ↓
IP address
```

For example:

```text
www.example.com
        ↓
93.184.215.14
```

DNS operates at **Layer 7 (Application Layer)** of the OSI model.

DNS normally uses:

```text
UDP port 53
```

and can also use:

```text
TCP port 53
```

---

## Important DNS Records

### A Record

An **A record** maps a hostname to an **IPv4 address**.

Example:

```text
example.com → 172.17.2.172
```

### AAAA Record

An **AAAA record** maps a hostname to an **IPv6 address**.

The important distinction is:

```text
A     → IPv4
AAAA  → IPv6
```

### CNAME Record

A **CNAME (Canonical Name)** record maps one domain name to another domain name.

Example:

```text
www.example.com → example.com
```

### MX Record

An **MX (Mail Exchange)** record identifies the mail server responsible for receiving email for a domain.

For example, when sending mail to:

```text
user@example.com
```

the sending mail system can use the domain's MX record to determine which mail server handles the email.

---

## DNS Lookup with nslookup

A common command-line DNS tool is:

```bash
nslookup www.example.com
```

Example output can contain both:

```text
A     → IPv4 address
AAAA  → IPv6 address
```

### Key answers

**Which DNS record type refers to IPv6?**

```text
AAAA
```

**Which DNS record type refers to the email server?**

```text
MX
```

---

# Task 3 — WHOIS: Who Owns a Domain?

## What is WHOIS?

WHOIS allows us to obtain registration information about a domain.

WHOIS is pronounced:

> "Who is"

It is **not an acronym**.

A WHOIS lookup can provide information such as:

- Domain name
- Registrar
- Creation date
- Updated date
- Expiration date
- Registrant information
- Registrar contact information
- Nameservers

The command-line tool can be used with:

```bash
whois example.com
```

An important cybersecurity use of WHOIS is **reconnaissance**.

For example, during reconnaissance, an analyst may investigate:

```text
Who registered the domain?
Who is the registrar?
When was the domain created?
When does it expire?
Which nameservers does it use?
```

Privacy protection can hide some registrant information.

---

# Task 4 — HTTP(S): Accessing the Web

## HTTP

**HTTP stands for Hypertext Transfer Protocol.**

It defines how web clients and web servers communicate.

For example:

```text
Browser
   ↓
HTTP request
   ↓
Web server
   ↓
HTTP response
   ↓
Browser
```

HTTP normally uses:

```text
TCP port 80
```

HTTPS normally uses:

```text
TCP port 443
```

HTTPS is HTTP protected using encryption mechanisms such as TLS.

---

## Common HTTP Methods

### GET

Used to retrieve information from a server.

Example:

```http
GET /index.html HTTP/1.1
```

### POST

Used to submit data to a server.

Examples include:

- Submitting a form
- Sending application data
- Uploading information

### PUT

Used to create or replace a resource.

### DELETE

Used to request deletion of a resource.

---

## Using Telnet to Understand HTTP

One of the useful exercises in this room was communicating with a web server manually using Telnet.

For example:

```bash
telnet 10.49.137.54 80
```

A basic HTTP request can look like:

```http
GET /flag.html HTTP/1.1
Host: anything
```

This demonstrates something important:

A browser is essentially automating these conversations for us.

Instead of manually typing:

```http
GET /index.html HTTP/1.1
Host: example.com
```

the browser constructs and sends the HTTP request automatically.

The server then returns an HTTP response.

---

## Important Lesson

The graphical browser interface hides the underlying protocol.

When we use:

```text
Firefox / Chrome
```

we don't normally see:

```text
GET
Host
HTTP/1.1
HTTP response
headers
```

But those exchanges are happening underneath.

This is one of the most important concepts I learned from this room.

---

# Task 5 — FTP: Transferring Files

## What is FTP?

**FTP stands for File Transfer Protocol.**

Unlike HTTP, which is primarily designed for web communication, FTP is specifically designed for transferring files.

FTP normally uses:

```text
TCP port 21
```

FTP uses a control connection and separate data connections for transferring information.

---

## Common FTP Commands

| Command | Purpose |
|---|---|
| `USER` | Provides username |
| `PASS` | Provides password |
| `LIST` | Lists files |
| `RETR` | Downloads a file |
| `STOR` | Uploads a file |

For example:

```bash
ftp MACHINE_IP
```

An anonymous FTP server may allow:

```text
Username: anonymous
Password: [blank]
```

After logging in:

```text
ls
```

can list available files.

To download a file:

```text
get flag.txt
```

---

## Important Lesson

FTP is designed specifically for file transfer, whereas HTTP is primarily designed for web resources.

The protocol commands are also visible when examining the traffic with tools such as Wireshark.

---

# Task 6 — SMTP: Sending Email

## What is SMTP?

**SMTP stands for Simple Mail Transfer Protocol.**

SMTP is responsible for **sending email**.

SMTP normally uses:

```text
TCP port 25
```

A simplified email flow is:

```text
Email client
     ↓
    SMTP
     ↓
Mail server
     ↓
Recipient's mail server
```

---

## Important SMTP Commands

### HELO / EHLO

Starts an SMTP session.

```text
HELO client.thm
```

or:

```text
EHLO client.thm
```

### MAIL FROM

Specifies the sender.

```text
MAIL FROM: <user@client.thm>
```

### RCPT TO

Specifies the recipient.

```text
RCPT TO: <user@server.thm>
```

### DATA

Tells the server that the email content is about to be sent.

```text
DATA
```

### Dot

A single period on its own line tells the SMTP server that the message is complete.

```text
.
```

### QUIT

Ends the SMTP session.

---

## SMTP Conversation

A simplified SMTP session looks like:

```text
HELO client.thm
MAIL FROM: <sender@example.com>
RCPT TO: <receiver@example.com>
DATA
From: sender@example.com
To: receiver@example.com
Subject: Test

Hello!
.
QUIT
```

---

# Task 7 — POP3: Receiving Email

## What is POP3?

**POP3 stands for Post Office Protocol version 3.**

POP3 is designed for retrieving email messages from a mail server.

POP3 normally uses:

```text
TCP port 110
```

The important distinction is:

```text
SMTP → sends email
POP3 → retrieves email
```

---

## Important POP3 Commands

| Command | Purpose |
|---|---|
| `USER` | Identifies the user |
| `PASS` | Provides password |
| `STAT` | Shows number and total size of messages |
| `LIST` | Lists messages and sizes |
| `RETR` | Retrieves a message |
| `DELE` | Marks a message for deletion |
| `QUIT` | Ends the session |

Example:

```text
USER linda
PASS Pa$$123
STAT
LIST
RETR 4
QUIT
```

The command:

```text
RETR 4
```

means:

> Retrieve message number 4.

---

## POP3 and Email Applications

POP3 is not a separate email platform.

It is a **protocol** used by email software to communicate with a mail server.

For example:

```text
Email application
       ↓
      POP3
       ↓
Mail server
```

---

# Task 8 — IMAP: Synchronising Email

## What is IMAP?

**IMAP stands for Internet Message Access Protocol.**

IMAP allows email clients to access and **synchronize** messages stored on a mail server.

IMAP normally uses:

```text
TCP port 143
```

---

## Why IMAP Exists

Suppose I have:

```text
Laptop
Phone
Desktop
```

and all three access the same mailbox.

With IMAP, the mailbox can remain on the server and its state can be synchronized between devices.

For example:

```text
                 Mail Server
                /     |      \
               /      |       \
           Laptop   Phone   Desktop
```

If an email is marked as read, moved, or deleted, that change can be synchronized.

---

## POP3 vs IMAP

### POP3

Focuses primarily on retrieving/downloading email.

```text
Server
  ↓
Email client
```

### IMAP

Keeps the mailbox synchronized with the server.

```text
              Mail Server
             /     |     \
          Laptop Phone Desktop
```

A useful way to remember them:

```text
SMTP → Send
POP3 → Retrieve
IMAP → Synchronize
```

---

## Important IMAP Commands

### LOGIN

Authenticates the user.

```text
LOGIN username password
```

### SELECT

Selects a mailbox/folder.

```text
SELECT inbox
```

### FETCH

Retrieves a message.

For example:

```text
FETCH 3 body[]
```

retrieves message 3.

Therefore, to retrieve the fourth message:

```text
FETCH 4 body[]
```

### MOVE

Moves messages to another mailbox.

### COPY

Copies messages to another mailbox.

### LOGOUT

Ends the IMAP session.

---

# How Email Applications Actually Use These Protocols

One of the most useful concepts from this room was understanding how protocols relate to applications such as Gmail, Outlook, and other email clients.

SMTP, POP3, and IMAP are **not separate email platforms**.

They are protocols that define how email systems communicate.

A simplified model is:

```text
                    EMAIL SYSTEM

        Sending
           │
           ▼
         SMTP
           │
           ▼
      Mail Server
           │
           │
     ┌─────┴─────┐
     ▼           ▼
   POP3         IMAP
     │           │
     ▼           ▼
 Download     Synchronize
     │           │
     └─────┬─────┘
           ▼
      Email Client
```

Modern email services may use additional security mechanisms such as TLS and modern authentication, but the underlying concepts remain important.

---

# Protocol and Port Cheat Sheet

| Protocol | Full Name | Purpose | Default Port |
|---|---|---|---:|
| **DNS** | Domain Name System | Domain → IP resolution | **53** |
| **HTTP** | Hypertext Transfer Protocol | Web communication | **80** |
| **HTTPS** | Hypertext Transfer Protocol Secure | Secure web communication | **443** |
| **FTP** | File Transfer Protocol | File transfer | **21** |
| **SMTP** | Simple Mail Transfer Protocol | Sending email | **25** |
| **POP3** | Post Office Protocol version 3 | Retrieving email | **110** |
| **IMAP** | Internet Message Access Protocol | Email synchronization | **143** |
| **TELNET** | Telecommunication Network | Remote terminal access | **23** |

---

# Important Commands Learned

## DNS

```bash
nslookup example.com
```

## WHOIS

```bash
whois example.com
```

## HTTP

```bash
telnet MACHINE_IP 80
```

Then:

```http
GET / HTTP/1.1
Host: anything
```

## FTP

```bash
ftp MACHINE_IP
```

Then:

```text
USER
PASS
LIST
RETR
STOR
```

## SMTP

```bash
telnet MACHINE_IP 25
```

Then:

```text
HELO
MAIL FROM
RCPT TO
DATA
.
QUIT
```

## POP3

```bash
telnet MACHINE_IP 110
```

Then:

```text
USER
PASS
STAT
LIST
RETR
QUIT
```

## IMAP

```bash
telnet MACHINE_IP 143
```

Then commands such as:

```text
LOGIN
SELECT
FETCH
MOVE
COPY
LOGOUT
```

---

# Key Concepts I Learned

### 1. Applications depend on protocols

A browser isn't the protocol itself.

For example:

```text
Browser
   ↓
HTTP/HTTPS
   ↓
Web server
```

Similarly:

```text
Email client
   ↓
SMTP / POP3 / IMAP
   ↓
Mail server
```

---

### 2. DNS makes the Internet easier to use

Instead of remembering:

```text
93.184.215.14
```

we can use:

```text
example.com
```

DNS performs the mapping.

---

### 3. Different DNS records have different purposes

```text
A      → IPv4
AAAA   → IPv6
CNAME  → Another domain name
MX     → Mail server
```

---

### 4. HTTP is a request/response protocol

A client sends a request:

```text
GET / HTTP/1.1
```

and the server sends a response.

This happens every time we access a web resource, even though the browser normally hides the details.

---

### 5. Email uses multiple protocols

There isn't one single protocol responsible for everything.

```text
SMTP → Send
POP3 → Retrieve
IMAP → Synchronize
```

---

### 6. Telnet can be useful for learning protocols

Although Telnet itself is insecure and should not be used for sensitive communications, it is useful in a controlled lab environment because it allows us to manually interact with text-based protocols.

Instead of relying completely on a graphical application, I can directly see the commands and responses exchanged between a client and server.

---

# Practical Security Perspective

Understanding these protocols is important in cybersecurity because network traffic can reveal a significant amount of information.

For example, when analyzing traffic with Wireshark, an analyst can identify:

- DNS queries
- HTTP requests
- FTP commands
- SMTP conversations
- POP3 authentication
- IMAP commands

Unencrypted protocols can potentially expose sensitive information.

For example, traditional POP3 over plain TCP can expose authentication information to someone capable of capturing the traffic.

This leads naturally into the next room:

**Networking Secure Protocols**

where the focus moves toward securing these types of communications.

---

# Conclusion

This room helped me understand what happens underneath common applications and services.

Before this room, protocols such as DNS, HTTP, FTP, SMTP, POP3, and IMAP could easily seem like isolated networking terms.

Now I can understand them as different communication systems serving different purposes:

```text
DNS
↓
Find the destination

HTTP / HTTPS
↓
Access web resources

FTP
↓
Transfer files

SMTP
↓
Send email

POP3
↓
Retrieve email

IMAP
↓
Synchronize email
```

The most important lesson for me was that **the applications we use every day are built on top of networking protocols**. Learning how to communicate with those protocols directly using tools such as `telnet`, `ftp`, `nslookup`, and `whois` makes it much easier to understand what is actually happening on the network.

This knowledge provides a foundation for understanding network traffic, troubleshooting, reconnaissance, packet analysis, and eventually network security.

---

## Room Completed

**TryHackMe:** Networking Core Protocols

**Main areas covered:**

- DNS
- WHOIS
- HTTP/HTTPS
- FTP
- SMTP
- POP3
- IMAP
- TCP ports
- Client/server communication
- Email protocols
- Network protocol analysis
- Manual protocol interaction