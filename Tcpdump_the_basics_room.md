# Tcpdump: The Basics

**Write-up by: Nicholas Kiyimba**

## Introduction

This TryHackMe room introduced the fundamentals of **tcpdump**, a command-line packet capture and analysis tool commonly used for network troubleshooting, traffic analysis, and security investigations.

While Wireshark provides a graphical interface for inspecting packets, tcpdump provides a fast and flexible command-line approach. This makes it particularly useful when working on remote Linux systems, servers, incident-response environments, or situations where a graphical interface is unavailable.

The room covered:

- Basic packet capture
- Network-interface selection
- Saving and reading packet captures
- Limiting packet counts
- Numeric address and port output
- Protocol, host, and port filtering
- Logical filter operators
- Packet-size filtering
- TCP flag filtering
- Binary operations
- Link-layer/MAC address inspection
- ASCII and hexadecimal packet display
- Combining tcpdump with Linux shell tools such as `wc`

---

# Task 1 — Introduction

The room began by introducing tcpdump as a command-line utility for capturing and inspecting network traffic.

A packet capture contains individual network packets observed by a network interface. Instead of manually examining every packet, tcpdump allows an analyst to filter traffic and display only the packets relevant to an investigation.

The most important concept from the room is:

> **Capture or read the traffic, filter it, display the information you need, and use command-line tools to extract useful metrics.**

A basic command is:

```bash
tcpdump
```

Running tcpdump without arguments can confirm that the program is installed, but in practical analysis we normally specify an interface, capture file, filter, or display option.

---

# Task 2 — Basic Packet Capture

## 2.1 Selecting a Network Interface

Before capturing traffic, tcpdump needs to know which network interface should be monitored.

The `-i` option specifies the interface:

```bash
tcpdump -i INTERFACE
```

For example:

```bash
sudo tcpdump -i eth0
```

To capture on all available interfaces:

```bash
sudo tcpdump -i any
```

The available interfaces can be identified with:

```bash
ip address show
```

or the shorter form:

```bash
ip a s
```

Example output may contain:

```text
1: lo:
2: ens5:
```

Here:

- `lo` is the loopback interface.
- `ens5` is a network interface used for network communication.

The important lesson is that the interface name depends on the system. It might be `eth0`, `ens5`, `wlan0`, `wlo1`, or another name.

---

## 2.2 Saving Captured Packets

Captures can be saved to a file with `-w`:

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

This writes the captured packets to:

```text
capture.pcap
```

The `.pcap` extension is commonly used for packet capture files.

One important behavior is that when `-w` is used, tcpdump does not display the normal packet output on screen. Instead, it writes the captured packets to the file.

This is useful when:

- Traffic needs to be analyzed later.
- Another analyst needs the capture.
- The capture will be opened in Wireshark.
- A security investigation requires evidence to be preserved.

---

## 2.3 Reading a Packet Capture

Previously saved packets can be read using `-r`:

```bash
tcpdump -r capture.pcap
```

This is extremely useful because packet captures can be analyzed without generating new traffic.

For example:

```bash
tcpdump -r traffic.pcap
```

The same filtering techniques used during live capture can also be applied when reading a `.pcap` file.

---

## 2.4 Limiting the Number of Packets

The `-c` option specifies how many packets tcpdump should capture:

```bash
sudo tcpdump -i eth0 -c 10
```

This captures 10 packets and then stops.

Without `-c`, a live capture normally continues until it is manually stopped with:

```text
CTRL+C
```

Limiting the packet count is useful when testing a command or when only a small sample is required.

---

## 2.5 Numeric Output with `-n`

Tcpdump may attempt to resolve IP addresses into hostnames.

For example, instead of displaying:

```text
93.184.215.14
```

it may display a hostname.

The `-n` option disables IP address resolution:

```bash
tcpdump -n
```

This makes output faster and keeps addresses in numeric form.

The room's question asking which option displays addresses only in numeric format therefore relates to:

```bash
-n
```

---

## 2.6 Preventing Both Address and Port Resolution

The `-nn` option disables both:

- IP/hostname resolution
- Port/service-name resolution

Example:

```bash
tcpdump -nn
```

Instead of displaying:

```text
22
```

as:

```text
ssh
```

tcpdump keeps the numeric port number.

This is particularly useful during analysis because numeric output is predictable and avoids unnecessary DNS/service-name lookups.

---

## 2.7 Verbose Output

Tcpdump supports different levels of verbosity.

```bash
-v
```

provides more packet information.

```bash
-vv
```

provides even more detail.

```bash
-vvv
```

provides an even more verbose decode.

The general idea is:

```text
normal
   ↓
-v
   ↓
-vv
   ↓
-vvv
```

More verbosity can be useful when investigating protocol fields, although excessive output can make large captures harder to read.

---

# Task 3 — Filtering Expressions

Capturing all network traffic is often impractical.

A host can generate a large amount of traffic, so an analyst needs to specify exactly what should be inspected.

Tcpdump uses **capture/display filter expressions** to accomplish this.

---

## 3.1 Filtering by Host

To capture traffic involving a particular host:

```bash
tcpdump host 192.168.1.10
```

This includes packets:

- from the host
- to the host

A hostname can also be used:

```bash
tcpdump host example.com
```

---

## 3.2 Filtering by Source Host

To capture packets originating from a specific host:

```bash
tcpdump src host 192.168.1.10
```

This means:

> Only packets whose source host is `192.168.1.10`.

---

## 3.3 Filtering by Destination Host

To capture packets sent to a specific host:

```bash
tcpdump dst host 192.168.1.10
```

This means:

> Only packets whose destination host is `192.168.1.10`.

The distinction is:

```text
host X
    ↓
traffic to OR from X

src host X
    ↓
traffic originating from X

dst host X
    ↓
traffic going to X
```

---

# 3.4 Filtering by Port

Ports can also be used as filters.

For example:

```bash
tcpdump port 53
```

This captures traffic involving port 53.

DNS commonly uses:

```text
UDP 53
TCP 53
```

Therefore:

```bash
tcpdump port 53
```

can be used to inspect DNS-related traffic.

---

## 3.5 Source and Destination Ports

Source port:

```bash
tcpdump src port 53
```

Destination port:

```bash
tcpdump dst port 53
```

The distinction is important when investigating the direction of a connection.

---

# 3.6 Filtering by Protocol

Tcpdump can filter by protocol.

Examples:

```bash
tcpdump tcp
```

```bash
tcpdump udp
```

```bash
tcpdump icmp
```

```bash
tcpdump ip
```

```bash
tcpdump ip6
```

For example:

```bash
sudo tcpdump -i ens5 icmp -n
```

captures ICMP packets while keeping addresses in numeric form.

ICMP echo request and echo reply traffic can indicate the use of `ping`.

ICMP time-exceeded messages can also appear during tools such as `traceroute`.

---

# 3.7 Logical Operators

Tcpdump supports logical operators that allow multiple conditions to be combined.

## `and`

Both conditions must be true.

```bash
tcpdump host 1.1.1.1 and tcp
```

This means:

> TCP traffic involving `1.1.1.1`.

---

## `or`

Either condition can be true.

```bash
tcpdump udp or icmp
```

This captures UDP or ICMP traffic.

---

## `not`

Excludes a condition.

```bash
tcpdump not tcp
```

This captures traffic that is not TCP.

A useful mental model is:

```text
and → BOTH
or  → EITHER
not → EXCLUDE
```

---

# 3.8 Combining Multiple Filters

The real power of tcpdump comes from combining conditions.

For example:

```bash
tcpdump -i eth0 host example.com and tcp port 443
```

This means:

```text
Interface:
eth0

Host:
example.com

Protocol:
TCP

Port:
443
```

In practical terms, this isolates TCP/443 traffic involving the specified host.

---

# 3.9 Counting Filtered Packets

Tcpdump output can be passed to other Linux commands using the pipe operator:

```text
|
```

For example:

```bash
tcpdump -r traffic.pcap icmp -n | wc -l
```

The process is:

```text
traffic.pcap
     ↓
 tcpdump
     ↓
ICMP packets
     ↓
    |
     ↓
 wc -l
     ↓
number of matching lines
```

This is an important Linux command-line workflow.

Instead of simply viewing packets, I can use tcpdump to produce a measurable result.

---

# 3.10 Reading DNS Queries

When inspecting DNS traffic, a packet might contain output similar to:

```text
192.168.139.132.47902 > 192.168.139.2.53:
47108+ A? example.org. (29)
```

The important section is:

```text
A? example.org.
```

The `A?` indicates an IPv4 address query.

For IPv6, a DNS query may appear as:

```text
AAAA? example.org.
```

Therefore, when asked to identify the hostname/subdomain appearing in a DNS query, I need to look at the text immediately associated with the DNS query, rather than assuming the IP addresses are the hostname.

---

# Task 4 — Advanced Filtering

Task 4 introduced more advanced packet filtering.

---

## 4.1 Filtering by Packet Size

Tcpdump supports size-based filters.

```bash
greater LENGTH
```

filters packets greater than or equal to the specified size.

For example:

```bash
tcpdump greater 15000
```

Similarly:

```bash
tcpdump less 15000
```

filters packets less than or equal to the specified size.

These filters can be combined with protocol, host, and port filters.

For example:

```bash
tcpdump tcp and greater 15000
```

This focuses on large TCP packets.

---

# 4.2 Binary Operations

Task 4 introduced three important binary operations.

## AND — `&`

```text
1 & 1 = 1
1 & 0 = 0
0 & 1 = 0
0 & 0 = 0
```

The result is `1` only when both input bits are `1`.

---

## OR — `|`

```text
1 | 1 = 1
1 | 0 = 1
0 | 1 = 1
0 | 0 = 0
```

The result is `1` when at least one input bit is `1`.

---

## NOT — `!`

NOT reverses a bit:

```text
!1 = 0
!0 = 1
```

These operations become important when tcpdump examines individual bits inside protocol headers.

---

# 4.3 Header Byte Access

Tcpdump allows individual bytes within protocol headers to be referenced using:

```text
proto[expr:size]
```

Where:

- `proto` identifies the protocol.
- `expr` represents the byte offset.
- `size` specifies how many bytes to inspect.

For example:

```text
tcp[tcpflags]
```

references the TCP flags field.

The room also demonstrated examples such as:

```text
ether[0] & 1 != 0
```

and:

```text
ip[0] & 0xf != 5
```

These demonstrate how tcpdump can perform bit-level analysis.

The important lesson is not to memorize these complex examples immediately, but to understand that tcpdump can inspect specific bytes and bits within packet headers.

---

# 4.4 TCP Flags

TCP contains several flags that describe the state and purpose of TCP segments.

The room covered:

```text
tcp-syn
tcp-ack
tcp-fin
tcp-rst
tcp-push
```

Some of their purposes are:

### SYN

Used when establishing a TCP connection.

### ACK

Acknowledges received TCP data or connection state.

### FIN

Used when gracefully closing a TCP connection.

### RST

Resets/aborts a TCP connection.

### PUSH

Requests that TCP data be pushed to the receiving application.

---

# 4.5 Exact TCP Flag Matching

To capture TCP packets where **only SYN is set**:

```bash
tcpdump "tcp[tcpflags] == tcp-syn"
```

The equality operator:

```text
==
```

requires the TCP flags field to match the specified condition.

For RST:

```bash
tcpdump "tcp[tcpflags] == tcp-rst"
```

This isolates packets where RST is the only TCP flag set.

---

# 4.6 Checking Whether a Flag Is Present

To capture packets where SYN is set, even if other flags are also present:

```bash
tcpdump "tcp[tcpflags] & tcp-syn != 0"
```

This is different from:

```bash
tcpdump "tcp[tcpflags] == tcp-syn"
```

The difference is:

```text
== tcp-syn
    ↓
SYN ONLY

& tcp-syn != 0
    ↓
SYN IS PRESENT
```

This distinction is important for accurate packet analysis.

---

# 4.7 Combining TCP Flags

Tcpdump can also check multiple flags.

For example:

```bash
tcpdump "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"
```

This checks whether SYN or ACK is set.

This demonstrates how bitwise operations can be used to build precise filters.

---

# Task 5 — Displaying Packets

Task 5 focused on changing how tcpdump displays packets.

Filtering determines **which packets** are examined.

Display options determine **how those packets are shown**.

---

# 5.1 Quick Output — `-q`

```bash
tcpdump -q
```

The `-q` option provides a shorter packet summary.

Instead of showing detailed TCP information, the output may look like:

```text
IP 104.18.12.149.https > g5000.45248: tcp 25
```

This is useful when scanning large amounts of traffic quickly.

---

# 5.2 Link-Level Header — `-e`

```bash
tcpdump -e
```

The `-e` option displays link-layer information.

On Ethernet networks this includes:

- Source MAC address
- Destination MAC address
- Ethernet type
- Ethernet frame length

For example:

```text
44:df:65:d8:fe:6c > 02:83:1e:40:5d:17
```

The address before `>` is the source MAC address.

The address after `>` is the destination MAC address.

This makes `-e` particularly useful when investigating:

- ARP
- DHCP
- Local Ethernet traffic
- Device identities
- Suspicious traffic at Layer 2

---

# 5.3 ASCII Output — `-A`

```bash
tcpdump -A
```

The `-A` option displays packet contents as ASCII where possible.

ASCII is useful when packet data contains readable text.

For example, plaintext HTTP traffic may contain readable information such as:

```text
GET /index.html HTTP/1.1
Host: example.com
```

However, encrypted or compressed data will generally not become readable simply by using `-A`.

---

# 5.4 Hexadecimal Output — `-xx`

```bash
tcpdump -xx
```

This displays packet data in hexadecimal.

Hexadecimal represents each byte using two hexadecimal digits.

For example:

```text
45
00
4d
fb
d8
```

This allows packet bytes to be examined at a low level.

---

# 5.5 Hexadecimal + ASCII — `-X`

```bash
tcpdump -X
```

This provides both hexadecimal and ASCII representations.

Conceptually:

```text
Hexadecimal                 ASCII
-----------------------------------------
4500 004d fbd8 ...          E..M...
```

This is useful because hexadecimal shows the raw bytes while ASCII helps identify readable portions of the packet.

---

# Task 6 — Conclusion / Mission Debrief

The final task reinforced the practical skills covered throughout the room.

The mission debrief confirmed several successful operations.

---

## Protocol Filtering + Counting

I successfully filtered ICMP packets and counted the resulting output using a shell pipeline.

The workflow was:

```text
Read ICMP packets from PCAP
        ↓
Apply -n numeric filtering
        ↓
Pipe output to wc
        ↓
Count matching packets
```

A representative command is:

```bash
tcpdump -r traffic.pcap icmp -n | wc -l
```

This demonstrates how tcpdump can be combined with standard Linux utilities to extract metrics from packet captures.

---

## TCP RST Filtering + Counting

I also used TCP flag filtering to isolate packets with only the RST flag set.

The important syntax was:

```bash
tcpdump -r traffic.pcap "tcp[tcpflags] == tcp-rst" -n | wc -l
```

The logic is:

```text
tcp[tcpflags]
      ↓
Look at TCP flags
      ↓
== tcp-rst
      ↓
Only RST is set
      ↓
wc -l
      ↓
Count the matches
```

This type of filtering can be useful when investigating:

- Connection failures
- Unexpected connection termination
- Reset patterns
- Potential network or application problems

---

# Lessons From the Improvement Feedback

The room also identified an error during the exercise:

```text
ICMP
```

was rejected as a protocol filter while:

```text
icmp
```

worked.

The important lesson is that tcpdump filter keywords need to be written using the correct syntax and case.

Instead of repeatedly guessing when a command fails, a better practice is to consult the documentation:

```bash
man tcpdump
```

and:

```bash
man pcap-filter
```

This is an important professional habit. Network analysts do not need to memorize every possible filter expression.

They need to understand the filtering logic and know how to quickly reference the correct syntax.

---

# Action Point 1 — Size-Based Filtering

The room recommended expanding size-based filtering.

Basic examples include:

```bash
tcpdump greater 15000
```

and:

```bash
tcpdump less 15000
```

These can be combined with other conditions.

For example:

```bash
tcpdump tcp and greater 15000
```

This allows an analyst to narrow a capture to larger TCP packets.

A more specific investigation could combine protocol, port, and size:

```bash
tcpdump tcp port 443 and greater 1000
```

The general principle is:

> **The more accurately the filter describes the traffic of interest, the less irrelevant traffic the analyst has to inspect.**

---

# Action Point 2 — Mastering Link-Layer Headers

The room also recommended becoming comfortable with:

```bash
-e
```

This is important because MAC addresses belong to the link layer.

For example:

```bash
tcpdump -r traffic.pcap -e arp -n
```

can be used to inspect ARP traffic while displaying the Ethernet addresses.

This is particularly useful for understanding local-network communication.

When an ARP request appears, the source MAC address identifies the device that transmitted the Ethernet frame.

---

# Key Commands Reference

## Capture Traffic

```bash
sudo tcpdump -i eth0
```

## Capture on All Interfaces

```bash
sudo tcpdump -i any
```

## Capture a Fixed Number of Packets

```bash
sudo tcpdump -i eth0 -c 10
```

## Save a Capture

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

## Read a Capture

```bash
tcpdump -r capture.pcap
```

## Numeric IP Addresses

```bash
tcpdump -n
```

## Numeric IP Addresses and Ports

```bash
tcpdump -nn
```

## Verbose Output

```bash
tcpdump -v
tcpdump -vv
tcpdump -vvv
```

## Host Filtering

```bash
tcpdump host 192.168.1.10
```

## Source Host

```bash
tcpdump src host 192.168.1.10
```

## Destination Host

```bash
tcpdump dst host 192.168.1.10
```

## Port Filtering

```bash
tcpdump port 53
```

## Source Port

```bash
tcpdump src port 53
```

## Destination Port

```bash
tcpdump dst port 53
```

## Protocol Filtering

```bash
tcpdump tcp
tcpdump udp
tcpdump icmp
```

## Logical Operators

```bash
tcpdump tcp and port 443
tcpdump udp or icmp
tcpdump not tcp
```

## Size Filtering

```bash
tcpdump greater 15000
tcpdump less 15000
```

## TCP Flag Filtering

```bash
tcpdump "tcp[tcpflags] == tcp-syn"
```

```bash
tcpdump "tcp[tcpflags] == tcp-rst"
```

```bash
tcpdump "tcp[tcpflags] & tcp-syn != 0"
```

## Quick Output

```bash
tcpdump -q
```

## Link-Layer Information

```bash
tcpdump -e
```

## ASCII

```bash
tcpdump -A
```

## Hexadecimal

```bash
tcpdump -xx
```

## Hex + ASCII

```bash
tcpdump -X
```

## Count Results

```bash
tcpdump ... | wc -l
```

---

# Practical Investigation Workflow

A useful way to approach tcpdump analysis is:

```text
1. Identify the capture/interface
              ↓
2. Decide what traffic matters
              ↓
3. Build a filter
              ↓
4. Add numeric output (-n/-nn)
              ↓
5. Add display options if necessary
              ↓
6. Inspect the packets
              ↓
7. Pipe results into Linux tools when metrics are required
```

For example:

```bash
tcpdump -r traffic.pcap -nn 'tcp port 443' | wc -l
```

This asks:

> How many TCP packets involving port 443 are present in the capture?

Another example:

```bash
tcpdump -r traffic.pcap -e arp -n
```

asks:

> What ARP traffic exists, and what Ethernet/MAC addresses are involved?

This is much more powerful than simply running tcpdump without filters.

---

# Key Takeaways

The most important concepts I gained from this room are:

1. **Tcpdump is a command-line packet capture and analysis tool.**
2. `-i` selects the network interface.
3. `-w` saves packets to a capture file.
4. `-r` reads an existing capture.
5. `-c` limits the number of captured packets.
6. `-n` prevents IP address resolution.
7. `-nn` prevents both IP and port/service resolution.
8. Host, port, and protocol filters reduce irrelevant traffic.
9. `and`, `or`, and `not` allow filters to be combined.
10. Packet-size filters can isolate unusually large or small packets.
11. `tcp[tcpflags]` allows TCP flag analysis.
12. `== tcp-rst` can isolate packets where RST is the only flag set.
13. `-e` exposes link-layer information such as MAC addresses.
14. `-A` displays packet data as ASCII.
15. `-xx` displays packet data in hexadecimal.
16. `-X` displays hexadecimal and ASCII together.
17. The Linux pipe operator `|` allows tcpdump output to be processed by other commands.
18. `wc -l` can be used to count matching tcpdump output.
19. Correct filter syntax and case matter.
20. `man tcpdump` and `man pcap-filter` are valuable references when constructing unfamiliar filters.

---

# Final Reflection

This room moved my packet-analysis skills from purely graphical analysis toward the command line.

The biggest lesson was not simply memorizing tcpdump options. It was learning how to construct an investigation from smaller components.

For example:

```bash
tcpdump -r traffic.pcap "tcp[tcpflags] == tcp-rst" -n | wc -l
```

combines several concepts:

- Reading a PCAP
- Filtering TCP
- Accessing the TCP flags field
- Performing an exact flag comparison
- Avoiding address resolution
- Piping command output
- Counting the resulting packets

That demonstrates a much more practical approach to network analysis than simply viewing packets one by one.

Tcpdump also complements Wireshark well. Wireshark provides a powerful graphical environment for detailed packet inspection, while tcpdump provides a lightweight command-line method for quickly capturing, filtering, and quantifying network traffic.

The next step is to continue building on these fundamentals by becoming faster at constructing filters and interpreting the network evidence they return.
