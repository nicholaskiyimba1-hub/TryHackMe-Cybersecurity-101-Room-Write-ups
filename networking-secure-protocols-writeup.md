# TryHackMe --- Networking Secure Protocols

## Overview

This room covers several common approaches used to secure network
communications:

1.  **TLS** --- secures application protocols such as HTTP, SMTP, POP3,
    and IMAP.
2.  **SSH** --- provides secure remote access and can also be used for
    secure file transfer and tunnelling.
3.  **SFTP and FTPS** --- secure alternatives for transferring files.
4.  **VPNs** --- create encrypted tunnels for connecting users, devices,
    or networks over the Internet.
5.  **Wireshark** --- used to inspect and analyze captured network
    traffic.

The main lesson is that different security mechanisms operate at
different layers and solve different problems.

------------------------------------------------------------------------

# 1. TLS

## What is TLS?

**TLS (Transport Layer Security)** is a cryptographic protocol used to
protect network communications.

It provides three important security properties:

-   **Confidentiality** --- prevents unauthorized parties from reading
    the traffic.
-   **Integrity** --- helps detect whether traffic has been modified.
-   **Authentication** --- certificates can be used to verify the
    identity of a server.

TLS is commonly used to secure existing application protocols.

For example:

  Insecure protocol   TLS-secured version
  ------------------- ----------------------------
  HTTP                HTTPS
  SMTP                SMTPS / SMTP with STARTTLS
  POP3                POP3S
  IMAP                IMAPS

A useful way to remember this is:

> **TLS is the security layer; the application protocol still performs
> the actual application function.**

------------------------------------------------------------------------

# 2. SSH

## What is SSH?

**SSH (Secure Shell)** is a protocol used primarily for secure remote
administration of systems.

The default SSH port is:

``` text
22/TCP
```

Unlike plaintext remote-access protocols such as Telnet, SSH encrypts
the communication between the client and server.

SSH can also provide:

-   Secure remote administration
-   Secure file transfer
-   Port forwarding
-   Tunnelling

Example:

``` bash
ssh username@hostname
```

------------------------------------------------------------------------

# 3. SFTP

## What is SFTP?

**SFTP (SSH File Transfer Protocol)** is a file-transfer protocol that
operates over SSH.

It should not be confused with FTPS.

SFTP normally uses:

``` text
22/TCP
```

because it operates through SSH.

Example connection:

``` bash
sftp username@hostname
```

Common SFTP commands include:

``` text
get filename
```

Downloads a file.

``` text
put filename
```

Uploads a file.

Other commands are similar to Unix shell commands, but SFTP has its own
command environment.

### SFTP vs FTPS

  Feature                 SFTP                 FTPS
  ----------------------- -------------------- -----------------------
  Security mechanism      SSH                  TLS
  Typical port            22                   990 for implicit FTPS
  Based on                SSH                  FTP + TLS
  Certificates required   No TLS certificate   Yes
  File transfer           Yes                  Yes

------------------------------------------------------------------------

# 4. FTPS

## What is FTPS?

**FTPS (FTP Secure)** is FTP protected using TLS.

FTPS should not be confused with SFTP.

FTP traditionally uses:

``` text
21/TCP
```

FTPS can use:

``` text
990/TCP
```

for implicit FTPS.

Because FTP uses separate control and data connections, FTPS can be more
complicated to configure through strict firewalls.

TLS certificates are required to establish secure TLS communication.

------------------------------------------------------------------------

# 5. Secure Protocol Port Mappings

A useful reference from the room:

  Protocol                       Common port
  ---------------------------- -------------
  FTP                                     21
  FTPS                                   990
  SSH                                     22
  Telnet                                  23
  SMTP                                    25
  SMTP Submission / STARTTLS             587
  HTTP                                    80
  HTTPS                                  443
  POP3                                   110
  POP3S                                  995
  IMAP                                   143
  IMAPS                                  993

Important pairings:

``` text
FTP       21  → FTPS      990
Telnet    23  → SSH       22
HTTP      80  → HTTPS     443
POP3      110 → POP3S     995
IMAP      143 → IMAPS     993
SMTP      25  → secure SMTP mechanisms such as STARTTLS/submission
```

------------------------------------------------------------------------

# 6. VPN

## What is a VPN?

A **VPN (Virtual Private Network)** creates a logical connection between
systems or networks across an existing network such as the Internet.

A VPN can allow geographically separated offices to communicate as
though they were connected through a private network.

The two important concepts are:

-   **Virtual** --- the connection is created using existing network
    infrastructure rather than a dedicated physical link.
-   **Private** --- traffic is protected through encryption and
    authentication mechanisms provided by the VPN technology.

------------------------------------------------------------------------

## Site-to-Site VPN

A company with multiple branches can establish a VPN between its
networks.

For example:

``` text
Branch A
   |
   | encrypted VPN tunnel
   |
Internet
   |
   | encrypted VPN tunnel
   |
Main Branch
```

This allows systems at the remote branch to access resources at the main
branch through the VPN.

This is commonly called a **site-to-site VPN**.

------------------------------------------------------------------------

## Remote-Access VPN

A VPN can also connect an individual device to a private network.

Example:

``` text
Laptop
   |
   | VPN tunnel
   |
Internet
   |
   |
Company VPN Server
   |
Private Company Network
```

This is commonly called a **remote-access VPN**.

------------------------------------------------------------------------

# 7. VPN and Public IP Addresses

When a VPN is configured to route Internet traffic through the VPN
server, external websites generally see the VPN server's public IP
address rather than the user's original public IP address.

For example:

``` text
User → VPN Server → Website
```

Instead of:

``` text
User → Website
```

This can make the user appear to be connecting from the VPN server's
location.

However, VPN behavior depends on configuration.

Some VPNs use **split tunnelling**, where only traffic intended for the
private network goes through the VPN while normal Internet traffic uses
the user's regular connection.

Therefore, connecting to a VPN does not automatically mean that every
packet is routed through it.

------------------------------------------------------------------------

# 8. VPN Limitations and Testing

A VPN should not automatically be assumed to provide complete privacy.

Depending on its configuration, you may need to test for:

-   IP address leaks
-   DNS leaks
-   IPv6 leaks
-   Split-tunnelling behavior
-   Incorrect routing

The important lesson is:

> **Never assume that a security control is working exactly as intended.
> Verify its behavior.**

------------------------------------------------------------------------

# 9. Wireshark Challenge

## Objective

The final challenge involved analyzing a packet capture containing TLS
traffic.

The provided files were:

``` text
randy-chromium.pcapng
ssl-key.log
```

The browser had been configured to save TLS session keys to the key-log
file.

Normally, TLS encrypts application data, meaning Wireshark cannot simply
display the contents of an HTTPS session.

However, when the appropriate TLS session keys are provided, Wireshark
can decrypt the captured TLS traffic.

------------------------------------------------------------------------

# 10. Loading the Packet Capture

The capture file was opened in Wireshark:

``` text
Documents/randy-chromium.pcapng
```

After opening the capture, Wireshark displayed the captured packets.

At first, the application data was protected by TLS.

------------------------------------------------------------------------

# 11. Configuring TLS Decryption

The TLS key-log file was configured in Wireshark.

The general procedure was:

``` text
Right-click a TLS packet
        ↓
Protocol Preferences
        ↓
Transport Layer Security
        ↓
Open Transport Layer Security Preferences
        ↓
Locate ssl-key.log
        ↓
Apply/OK
```

The key-log file was located in the `Documents` directory.

After configuring it, Wireshark could decrypt the captured TLS sessions
for which the keys were available.

------------------------------------------------------------------------

# 12. Finding the Interesting Traffic

Simply decrypting the traffic was not enough.

The capture contained many packets, so the next step was to identify the
application protocol.

The following display filter was useful:

``` text
http2
```

This revealed HTTP/2 traffic.

One of the packets contained a POST request associated with a Facebook
login page.

The important part of the decrypted request contained parameters similar
to:

``` text
email=...
&pass=...
```

This demonstrated why packet analysis requires more than simply looking
at packet numbers.

The analyst needs to:

1.  Identify the protocol.
2.  Filter the traffic.
3.  Locate interesting requests.
4.  Inspect the packet contents.
5.  Extract the relevant information.

------------------------------------------------------------------------

# 13. Challenge Answer

The decrypted login request contained:

``` text
pass=THM%7BB8WM6P%7D
```

The value was URL-encoded.

Decoding:

``` text
%7B → {
%7D → }
```

produced:

``` text
THM{B8WM6P}
```

**Answer:**

``` text
THM{B8WM6P}
```

------------------------------------------------------------------------

# 14. What I Learned

This was my first practical exposure to **Wireshark**, and initially the
interface looked complicated because a packet capture contains a large
amount of information.

The most important lesson was not memorizing every Wireshark menu. It
was learning how to approach a packet capture logically.

My investigation workflow was:

``` text
PCAP
 ↓
Identify the protocol
 ↓
Apply a display filter
 ↓
Find interesting traffic
 ↓
Inspect the packet
 ↓
Follow the application data
 ↓
Extract the required information
```

For this challenge:

``` text
randy-chromium.pcapng
        ↓
TLS decryption using ssl-key.log
        ↓
HTTP/2
        ↓
POST request
        ↓
Login data
        ↓
Password
```

The challenge also reinforced an important security concept:

> **Encryption protects captured traffic from being read without the
> required keys, but if an analyst legitimately possesses the
> appropriate session keys, encrypted traffic can potentially be
> decrypted and investigated.**

------------------------------------------------------------------------

# 15. Key Takeaways

### TLS

TLS provides encryption, integrity, and authentication for network
communications.

### SSH

SSH provides secure remote access and can also be used for file transfer
and tunnelling.

### SFTP

SFTP is a file-transfer protocol operating over SSH.

### FTPS

FTPS is FTP secured using TLS.

### VPN

VPNs create protected logical tunnels across existing networks and can
connect users or entire networks.

### Wireshark

Wireshark is a packet-analysis tool used to capture and inspect network
traffic.

Most importantly:

> **Don't try to understand every packet. Start with the question you
> are trying to answer, identify the relevant protocol, filter the
> traffic, and then inspect the interesting packets.**

------------------------------------------------------------------------

## Skills Practiced

-   TLS concepts
-   Secure network protocols
-   SSH
-   SFTP
-   FTPS
-   VPN fundamentals
-   TCP/UDP port identification
-   Wireshark packet analysis
-   TLS traffic decryption
-   HTTP/2 analysis
-   HTTP POST request inspection
-   URL encoding/decoding
-   Credential identification in a controlled lab environment

------------------------------------------------------------------------

## Tools Used

-   TryHackMe
-   Wireshark
-   Chromium
-   Linux
-   Packet Capture (`.pcapng`)
-   TLS key-log file (`ssl-key.log`)
