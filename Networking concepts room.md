# Networking Concepts

**Write-up by: Nicholas Kiyimba**

## Introduction

This is the first room in a four-room networking series (Networking Concepts → Networking Essentials → Networking Core Protocols → Networking Secure Protocols). This room covers the foundational theory: the OSI model, the TCP/IP model, IP addressing and subnets, routing, UDP vs TCP, encapsulation, and a hands-on look at using `telnet` to manually talk to servers over TCP.

Understanding these fundamentals matters in cybersecurity because almost everything that happens on a network — traffic analysis, packet captures, firewall rules, exploitation of network services — is described in terms of these exact layers and protocols. Without a solid grasp of OSI/TCP-IP and how data actually gets encapsulated and delivered, tools like Wireshark or concepts like "layer 3 vs layer 7" attacks don't make much sense.

## Task 1 — Introduction

### Explanation

This task laid out the room's learning objectives: the OSI model, IP addresses/subnets/routing, TCP, UDP, and port numbers, and how to connect to an open TCP port from the command line. It assumes familiarity with terms like IP address and port number, without requiring the ability to explain them in depth yet — that's what this room builds.

## Task 2 — OSI Model

### Explanation

The **OSI (Open Systems Interconnection) model** is a conceptual framework developed by the ISO to describe how network communication should occur. It's theoretical rather than something directly implemented, but understanding it is essential for making sense of networking terminology at a deeper level (e.g. what people mean by "layer 3 switch" or "layer 7 firewall").

The model has seven layers, numbered from the bottom (Layer 1) to the top (Layer 7):

| Layer | Name | Main Function | Example Protocols/Standards |
|---|---|---|---|
| 7 | Application | Provides services/interfaces to applications | HTTP, FTP, DNS, POP3, SMTP, IMAP |
| 6 | Presentation | Data encoding, encryption, compression | Unicode, MIME, JPEG, PNG, MPEG |
| 5 | Session | Establishing, maintaining, synchronising sessions | NFS, RPC |
| 4 | Transport | End-to-end communication, data segmentation | TCP, UDP |
| 3 | Network | Logical addressing and routing between networks | IP, ICMP, IPSec |
| 2 | Data Link | Reliable data transfer between adjacent nodes | Ethernet (802.3), WiFi (802.11) |
| 1 | Physical | Physical transmission medium | Electrical, optical, wireless signals |

Going through each layer in more depth:

- **Layer 1 (Physical)** – the actual physical connection: cables, radio signals, and the definition of how 0s and 1s are represented electrically, optically, or wirelessly.
- **Layer 2 (Data Link)** – defines how devices on the *same* network segment (e.g. a group of computers on one switch) agree to communicate. Ethernet and WiFi both operate here, and this is where MAC addresses live — six-byte hardware addresses, usually written in hex, where the leftmost three bytes identify the manufacturer. Every frame includes both a source and destination MAC address.
- **Layer 3 (Network)** – handles communication *between different* networks, using logical addressing (IP addresses) and routing to find a path between them. Examples include IP, ICMP, and VPN protocols like IPSec.
- **Layer 4 (Transport)** – provides end-to-end communication between applications running on different hosts, including things like flow control and error correction. TCP and UDP are the two examples here.
- **Layer 5 (Session)** – establishes, maintains, and synchronises communication sessions between applications, including negotiating session parameters and handling recovery if transmission fails. Examples: NFS, RPC.
- **Layer 6 (Presentation)** – makes sure data is delivered in a form the application layer can actually understand — handling encoding (like ASCII/Unicode), compression, and encryption. MIME (used to encode binary email attachments using 7-bit ASCII) is a good real-world example here.
- **Layer 7 (Application)** – provides network services directly to end-user applications, e.g. a web browser using HTTP to request a page. Examples: HTTP, FTP, DNS, and others.

A useful mnemonic for remembering the order bottom-to-top is "Please Do Not Throw Spinach Pizza Away" (Physical, Data Link, Network, Transport, Session, Presentation, Application).

### Questions

**Question:** Which layer is responsible for end-to-end communication between running applications?

**Answer:** The Transport layer (Layer 4).

**Question:** Which layer is responsible for routing packets to the proper network?

**Answer:** The Network layer (Layer 3).

**Question:** In the OSI model, which layer is responsible for encoding the application data?

**Answer:** The Presentation layer (Layer 6).

**Question:** Which layer is responsible for transferring data between hosts on the same network segment?

**Answer:** The Data Link layer (Layer 2).

## Task 3 — TCP/IP Model

### Explanation

The **TCP/IP model** is the practical, implemented counterpart to the theoretical OSI model. It was developed in the 1970s by the US Department of Defense, specifically designed so a network could keep functioning even if parts of it went down (e.g. during a military attack) — achieved partly through routing protocols that can adapt as the network's topology changes.

Where OSI has seven layers, TCP/IP condenses these into four (going top to bottom):

- **Application Layer** – combines OSI's Application, Presentation, and Session layers (layers 7, 6, and 5) into one
- **Transport Layer** – maps directly to OSI layer 4
- **Internet Layer** – maps to OSI's Network layer (layer 3); just a different name for the same concept
- **Link Layer** – maps to OSI's Data Link layer (layer 2)

| Layer # | OSI Model | TCP/IP Model | Protocols |
|---|---|---|---|
| 7 | Application | Application | HTTP, HTTPS, FTP, DNS, Telnet, etc. |
| 6 | Presentation | (same, folded in) | |
| 5 | Session | (same, folded in) | |
| 4 | Transport | Transport | TCP, UDP |
| 3 | Network | Internet | IP, ICMP, IPSec |
| 2 | Data Link | Link | Ethernet 802.3, WiFi 802.11 |
| 1 | Physical | (implicit) | |

Some modern networking textbooks (e.g. Kurose and Ross's *Computer Networking: A Top-Down Approach*) present a five-layer version instead, explicitly including the Physical layer: Application, Transport, Network, Link, Physical.

The rest of the room focuses specifically on the IP protocol (Internet layer) and TCP/UDP (Transport layer).

### Questions

**Question:** To which layer does HTTP belong in the TCP/IP model?

**Answer:** The Application layer.

**Question:** How many layers of the OSI model does the application layer in the TCP/IP model cover?

**Answer:** Three (the OSI Application, Presentation, and Session layers — layers 7, 6, and 5).

## Task 4 — IP Addresses and Subnets

### Explanation

Every host on a network needs a unique identifier so other hosts can reach it unambiguously — this is the role of an **IP address**, similar in concept to a home postal address. IPv4 is still the most common version (IPv6 being the other), and when "IP address" is mentioned without a version, IPv4 is generally assumed.

An IPv4 address is made up of four **octets** (32 bits total), each representing a decimal number from 0–255 (an octet being 8 bits). Within any given subnet, the all-zeros address (e.g. `192.168.1.0`) is reserved as the **network address**, and the all-255s address (e.g. `192.168.1.255`) is reserved as the **broadcast address** (used to send to every host on that network at once). Because of this, and the 32-bit limit, there are approximately 2³² (~4 billion) unique IPv4 addresses possible, minus reserved addresses.

**Checking network configuration from the command line:**

- Windows: `ipconfig`
- Linux/Unix: `ifconfig`, or the more modern `ip address show` (shorthand: `ip a s`)

Both show the assigned IP address, subnet mask, and broadcast address for each interface.

A subnet mask like `255.255.255.0` can also be written in CIDR notation as `/24`, meaning the leftmost 24 bits of the IP address stay fixed across the whole subnet (i.e. the first three octets are shared), leaving the last octet (256 values, minus network/broadcast) as the usable host range — for example, `192.168.66.1` through `192.168.66.254`.

**Private vs Public addresses:** Most IP addresses fall into one of two categories. Per RFC 1918, three specific ranges are reserved as **private** addresses, meant to never be directly reachable from the public internet:

- `10.0.0.0 – 10.255.255.255` (10/8)
- `172.16.0.0 – 172.31.255.255` (172.16/12)
- `192.168.0.0 – 192.168.255.255` (192.168/16)

A private address is like an internal numbering system inside a gated compound — everyone inside can reach each other, but nothing outside can reach in directly. For a device using a private address to reach the internet, the router in front of it needs a public IP address and must perform **Network Address Translation (NAT)**.

**Routing:** A router works like a local post office — it inspects a packet's destination IP address (working at Layer 3) and forwards it toward the best next network/router to get it closer to its final destination. A packet often passes through several routers before reaching its destination.

### Questions

**Question:** Which of the following IP addresses is not a private IP address? (`192.168.250.125`, `10.20.141.132`, `49.69.147.197`, `172.23.182.251`)

**Answer:** `49.69.147.197` — it doesn't fall within any of the three RFC 1918 private ranges, making it a public address.

**Question:** Which of the following IP addresses is not a valid IP address? (`192.168.250.15`, `192.168.254.17`, `192.168.305.19`, `192.168.199.13`)

**Answer:** `192.168.305.19` — `305` exceeds the maximum valid octet value of 255, making this an invalid IP address.

## Task 5 — UDP and TCP

### Explanation

Once a packet reaches the correct host (via its IP address), we need a way to reach the correct *process* running on that host — this is the role of **port numbers**, used by the two main Layer 4 transport protocols: UDP and TCP. A port number is two octets, giving a valid range of 1–65535 (port 0 is reserved).

**UDP (User Datagram Protocol)** is a simple, connectionless protocol — it doesn't establish a connection first, and doesn't even confirm that a packet was actually delivered. It's comparable to standard postal mail with no delivery confirmation: cheaper/faster, but with no guarantee of receipt.

**TCP (Transmission Control Protocol)** is connection-oriented and reliable — it uses mechanisms to guarantee data actually gets delivered correctly. Every octet of data sent gets a sequence number, letting the receiver detect lost or duplicated packets, and the receiver sends back an acknowledgement number confirming the last octet it received.

A TCP connection is established with a **three-way handshake**, using the SYN and ACK flags:

1. **SYN** – the client sends a SYN packet containing a randomly chosen initial sequence number
2. **SYN-ACK** – the server responds with its own randomly chosen sequence number, combined with an acknowledgement of the client's SYN
3. **ACK** – the client acknowledges the server's SYN-ACK, completing the handshake

Only after this handshake completes can actual data be exchanged over that TCP connection.

### Questions

**Question:** Which protocol requires a three-way handshake?

**Answer:** TCP.

**Question:** What is the approximate number of port numbers (in thousands)?

**Answer:** Approximately 65 thousand (65,535 valid port numbers).

## Task 6 — Encapsulation

### Explanation

**Encapsulation** is the process where each layer adds its own header (and sometimes a trailer) to the data it receives from the layer above, before passing the result down to the layer below. This lets each layer focus purely on its own job without needing to understand what's happening at any other layer.

Working through the layers for outgoing data:

1. **Application data** – starts as whatever the user actually inputs (e.g. writing an email or a search query), formatted according to whichever application protocol is being used
2. **Transport segment/datagram** – the transport layer (TCP or UDP) adds its header, producing a **TCP segment** or **UDP datagram**
3. **Network packet** – the Internet/Network layer adds an IP header, producing an **IP packet** that can actually be routed
4. **Data link frame** – the link layer (Ethernet or WiFi) adds its own header and trailer, producing a **frame**

This entire process runs in reverse on the receiving end, stripping headers back off one layer at a time until the original application data is recovered.

**The life of a packet (example: searching TryHackMe):**

1. I type a search query into TryHackMe's search box and hit enter
2. My browser (using HTTPS) builds an HTTP request and passes it down to the transport layer
3. TCP establishes a connection via the three-way handshake, then sends the HTTP request as one or more TCP segments, passed down to the Internet layer
4. The IP layer adds the source (my computer) and destination (TryHackMe's server) IP addresses, then passes the resulting packet down to the link layer
5. The link layer adds the appropriate header/trailer for the physical medium in use and sends the frame out to the router
6. Each router along the path strips the link-layer header, inspects the destination IP, and forwards the packet on toward the next hop, repeating until it reaches the destination network

The whole process then runs in reverse as the packet arrives at its destination.

### Questions

**Question:** On a WiFi, within what will an IP packet be encapsulated?

**Answer:** A WiFi (802.11) frame.

**Question:** What do you call the UDP data unit that encapsulates the application data?

**Answer:** A datagram.

**Question:** What do you call the data unit that encapsulates the application data sent over TCP?

**Answer:** A segment.

## Task 7 — Telnet

### Explanation

I started both the lab machine and the AttackBox, giving each a couple of minutes to boot, then opened a terminal on the AttackBox to work with `telnet`.

**TELNET** is a network protocol for remote terminal connections, and the `telnet` client lets me connect to and issue text commands to any server listening on a TCP port — not just for remote administration (its original purpose), but for manually talking to any TCP-based service to see how it responds.

I tested three services on the target machine:

```text
telnet MACHINE_IP 7
```

**Explanation:** Connects to the **Echo server** on port 7, which simply echoes back anything sent to it. This is a good way to directly observe a raw TCP round-trip. The connection is closed by pressing `Ctrl + ]`.

```text
telnet MACHINE_IP 13
```

**Explanation:** Connects to the **Daytime server** on port 13, which replies with the current date and time and then closes the connection automatically — no interaction needed beyond connecting.

Note: both the echo and daytime servers are considered security risks in real environments (they can be abused for reflection/amplification and information disclosure) and shouldn't normally be run — they were only started here specifically to demonstrate raw TCP communication.

```text
telnet MACHINE_IP 80
```

**Explanation:** Connects to the **web (HTTP) server** on port 80. Once connected, I manually typed an HTTP request:

```text
GET / HTTP/1.1
Host: telnet.thm
```

followed by pressing Enter twice, so the last line sent is blank (this blank line is what signals to the server that the request is complete). The server then responded with the raw HTTP response, starting with a status line like `HTTP/1.1 200 OK` followed by response headers and the page content — this is genuinely useful for understanding that a web browser is really just automating this exact same conversation over TCP, just with a GUI wrapped around it.

### Questions

**Question:** Use `telnet` to connect to the web server on MACHINE_IP. What is the name and version of the HTTP server?

**Answer:** *This is shown in the `Server:` header of the HTTP response from your lab instance, which I don't have recorded. Let me know what it returned and I'll add it here.*

**Question:** What flag did you get when you viewed the page?

**Answer:** *This is specific to the page content returned on your lab instance — I don't have that recorded. Let me know the flag and I'll add it here.*

## Task 8 — Conclusion

### Explanation

This room covered the OSI and TCP/IP models side by side, IP addressing and subnetting, a brief look at routing, TCP and UDP, encapsulation, and a hands-on demonstration using `telnet` to talk directly to servers over TCP. The next room in the series is Networking Essentials.

## Key Takeaways

- Learned the seven OSI layers, their functions, and example protocols at each, along with the "Please Do Not Throw Spinach Pizza Away" mnemonic.
- Understood how the four/five-layer TCP/IP model maps onto the OSI model, and that OSI's top three layers collapse into a single Application layer in TCP/IP.
- Learned how IPv4 addressing, octets, network/broadcast addresses, and CIDR notation (`/24`) work, plus the three RFC 1918 private address ranges and why NAT is needed for private addresses to reach the internet.
- Understood the core difference between UDP (connectionless, no delivery guarantee) and TCP (connection-oriented, reliable, using the three-way SYN/SYN-ACK/ACK handshake).
- Learned how encapsulation wraps data with a new header (and sometimes trailer) at each layer going down the stack, producing segments/datagrams, packets, and frames in turn.
- Used `telnet` hands-on to talk directly to an echo server, a daytime server, and a raw HTTP server, seeing exactly what a "real" protocol exchange looks like underneath the abstractions a browser normally hides.

## Conclusion

This room built a genuinely solid theoretical foundation that I can already tell will keep coming up: OSI/TCP-IP layering, IP addressing, and TCP vs UDP are the vocabulary that basically all further networking and security topics get described in. The `telnet` exercises were the most useful part practically, since actually typing a raw HTTP request by hand made the concept of "layers" and encapsulation feel concrete rather than abstract. I'm looking forward to building on this in the Networking Essentials room next.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
