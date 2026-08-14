# Wireshark: The Basics — TryHackMe Write-Up

## Overview

This write-up documents my completion of the **Wireshark: The Basics** room on TryHackMe.

This was my first practical experience with Wireshark. The purpose of the room was to introduce the Wireshark interface, packet analysis, packet navigation, filtering, and basic investigation of network traffic captures.

Wireshark is an open-source, cross-platform network packet analyzer that can capture live network traffic and analyze previously captured traffic stored in files such as `.pcap` and `.pcapng`.

---

## Learning Objectives

By completing this room, I learned how to:

- Navigate the Wireshark interface.
- Open and inspect packet capture files.
- Understand the different packet-analysis panes.
- Perform basic packet dissection.
- Relate packet information to OSI layers.
- Navigate to specific packets.
- Search packet contents.
- Mark and comment on packets.
- Extract transferred files from packet captures.
- Use Wireshark Expert Information.
- Apply basic display filters.
- Follow network streams.
- Inspect HTTP requests and responses.
- Extract useful information from network traffic.

---

# 1. Wireshark Interface

When Wireshark is opened, several important sections are available.

### Main Components

| Component | Purpose |
|---|---|
| Toolbar | Provides menus and shortcuts for capturing, filtering, sorting and processing traffic. |
| Display Filter Bar | Used to create and apply display filters. |
| Recent Files | Shows recently opened capture files. |
| Capture Interfaces | Shows available network interfaces for live packet capture. |
| Status Bar | Displays information such as packet counts and capture status. |

### Packet Analysis Panes

After opening a capture file, Wireshark presents packet information using three main panes:

**Packet List Pane**

Displays a summary of packets, including packet number, source, destination, protocol and packet information.

**Packet Details Pane**

Shows the protocols and fields contained within the selected packet.

**Packet Bytes Pane**

Displays the raw packet data in hexadecimal and ASCII representation.

---

# 2. Capture Files

The room provided two capture files:

- `http1.pcapng`
- `Exercise.pcapng`

`http1.pcapng` was used to reproduce the examples and screenshots shown in the room.

`Exercise.pcapng` was used to answer the practical questions.

This distinction was important because the exercise questions were based on the `Exercise.pcapng` capture.

---

# 3. Capture File Properties

Wireshark provides information about the capture file through:

**Statistics → Capture File Properties**

This allows an analyst to inspect information such as:

- Capture file comments
- Number of packets
- Capture time
- Capture interfaces
- File information
- File hashes

The SHA256 hash can be useful for identifying a capture file and verifying that the file being analyzed is the expected file.

### Questions

**Screenshot simulation file:**

`http1.pcapng`

**Question-answering file:**

`Exercise.pcapng`

The remaining answers were obtained directly from the `Exercise.pcapng` file during the lab.

---

# 4. Packet Dissection

Packet dissection is the process of decoding the protocols and fields contained within a packet.

I investigated **packet number 38** and examined its protocol layers.

A packet can contain several protocol layers corresponding to different parts of the OSI model.

A simplified representation is:

```text
Frame
  ↓
Ethernet
  ↓
IP
  ↓
TCP
  ↓
HTTP
  ↓
Application Data
```

### Important packet fields

**Frame**

Contains information about the captured frame, including arrival information.

**Ethernet**

Contains Layer 2 information such as source and destination MAC addresses.

**IP**

Contains Layer 3 information such as source IP, destination IP and TTL.

**TCP**

Contains Layer 4 information including source and destination ports and TCP-related fields.

**HTTP**

Contains application-layer information such as HTTP headers and application data.

---

## Packet 38 Investigation

For packet 38, I investigated:

- The markup language used under HTTP.
- The packet arrival date.
- The IP TTL value.
- The TCP payload size.
- The HTTP E-Tag value.

The important lesson was learning to locate information according to the protocol layer where it belongs.

For example:

```text
Arrival information → Frame
TTL                  → IP
TCP payload          → TCP
E-Tag                → HTTP
```

This is a useful way of approaching packet-analysis questions:

> Identify the information being requested → determine which protocol contains it → expand that protocol → inspect the relevant field.

---

# 5. Packet Navigation

Wireshark assigns a unique number to every packet in a capture.

For example:

```text
Packet 1
Packet 2
Packet 3
Packet 4
...
Packet 33790
```

Packet numbers make it easier to return to specific events during an investigation.

Wireshark provides **Go to Packet** functionality for quickly navigating to a specific packet.

---

## Finding Packets

Wireshark can also search inside packet data.

The **Edit → Find Packet** functionality supports different search types, including:

- Display filter
- Hex
- String
- Regular expression

The search can also be performed against different packet panes.

This is important because information visible in the Packet Details pane may not be present in the Packet List pane.

---

## Packet Marking

Packets can be marked so that they can easily be identified later.

Marked packets are displayed in black.

Packet marking is useful when an analyst wants to identify packets that require further investigation.

---

## Packet Comments

Comments can be attached to individual packets.

This allows analysts to document:

- Suspicious packets
- Important events
- Findings
- Notes for future investigation

Unlike temporary packet marking, packet comments can remain stored in the capture file.

---

# 6. Exporting Packets and Objects

Large packet captures can contain thousands or even millions of packets.

Sometimes an analyst only needs a specific portion of the capture.

Wireshark can export selected packets for further analysis.

It can also extract certain transferred objects from supported protocols.

For example, files transferred through HTTP can be reconstructed and exported from the capture.

The general investigation process is:

```text
PCAP
 ↓
Network traffic
 ↓
Protocol stream
 ↓
Transferred object
 ↓
Export file
 ↓
Further investigation
```

This is particularly useful during network-forensics investigations.

---

# 7. Time Display

Wireshark can display packet timestamps in different formats.

The default display represents time relative to the beginning of the capture.

The time display can be changed through:

**View → Time Display Format**

Changing the time format can make it easier to understand when events occurred during an investigation.

---

# 8. Expert Information

Wireshark has an **Expert Information** feature that identifies potentially interesting protocol conditions.

It categorizes information according to severity.

| Severity | Meaning |
|---|---|
| Chat | Normal protocol workflow information |
| Note | Notable events |
| Warn | Potential problems or unusual conditions |
| Error | More serious protocol problems |

Expert Information is useful for quickly identifying areas that may require investigation.

However, it should not be treated as proof that something malicious occurred.

An analyst must investigate the underlying packets because Expert Information can produce false positives and false negatives.

---

# 9. Packet Filtering

One of the most important Wireshark features is its filtering system.

There are two major types of filters:

### Capture Filters

Capture filters determine which packets are captured during a live capture.

```text
Network traffic
      ↓
Capture filter
      ↓
Only matching packets captured
```

### Display Filters

Display filters operate on packets that have already been captured.

```text
Existing capture
      ↓
Display filter
      ↓
Only matching packets displayed
```

A display filter does **not** delete the packets that don't match.

---

# 10. Apply as Filter

A useful Wireshark feature is:

**Right-click → Apply as Filter**

This allows Wireshark to automatically construct a display-filter query based on a selected packet field.

This is much easier than manually writing every filter.

A useful rule from the room was:

> If you can click on it, you can filter and copy it.

---

# 11. Conversation Filter

A normal filter can focus on one particular field.

A **Conversation Filter** allows an analyst to focus on related traffic between communicating hosts.

For example:

```text
Client IP + Client Port
        ↕
Server IP + Server Port
```

This is useful when investigating a particular communication session.

---

# 12. Colourise Conversation

Colourising a conversation highlights related packets without removing other packets from the packet list.

This is different from applying a display filter.

### Filter

```text
Unrelated packets → hidden
Related packets   → displayed
```

### Colourise

```text
Unrelated packets → remain visible
Related packets   → highlighted
```

This makes it easier to visually follow a particular conversation.

---

# 13. Prepare as Filter

**Prepare as Filter** creates a filter expression without immediately applying it.

This is useful when constructing more complex filters using logical conditions such as:

```text
AND
OR
```

It provides more control when building a query before executing it.

---

# 14. Apply as Column

Wireshark allows packet fields to be added as columns to the Packet List pane.

This can be useful when an analyst wants to compare a specific field across many packets.

For example, instead of repeatedly opening packets to inspect a field, the field can be displayed directly in the packet list.

---

# 15. Follow HTTP Stream

One of the most useful features I learned was **Follow Stream**.

Individual network packets only contain portions of a communication.

Following a stream reconstructs the communication so that it can be viewed at the application level.

For example:

```text
Client
  │
  │ HTTP Request
  ▼
Server
  │
  │ HTTP Response
  ▼
Client
```

This makes it much easier to understand what was actually communicated.

It can also expose unencrypted application data transmitted through protocols such as HTTP.

After following a stream, Wireshark automatically creates a display filter for that particular conversation.

---

# 16. Basic Display Filters

Wireshark supports many display-filter queries.

### Filter by protocol

For example:

```text
http
```

This displays HTTP traffic.

Other protocols can be filtered using their protocol names, such as:

```text
arp
dhcp
ftp
smtp
pop
imap
```

### Filter by port

A protocol can also be filtered using its port.

For example:

```text
tcp.port == 80
```

### Filter by IP address

A specific IP address can be filtered using:

```text
ip.addr == 192.168.1.2
```

These filters make it possible to reduce large packet captures to the traffic relevant to an investigation.

---

# 17. Practical Investigation

During the practical exercises, I used `Exercise.pcapng` to perform several investigations.

### Search for a String

I searched for:

```text
r4w
```

within the **Packet Details** pane and investigated the packet containing the string.

This demonstrated how Wireshark can search packet contents rather than only packet numbers.

---

### Packet 12

I navigated directly to:

```text
Packet 12
```

and inspected its packet comments.

This demonstrated how analysts can attach and retrieve investigation notes associated with individual packets.

---

### Extracting a TXT File

The capture contained a `.txt` file transferred through the network.

I used Wireshark's object-export functionality to locate and extract the file.

The extracted file could then be examined independently.

For example, on Linux:

```bash
cat filename.txt
```

A file hash can also be calculated with:

```bash
md5sum filename.txt
```

---

### Expert Information

I opened the Expert Information section and investigated the number of warnings reported by Wireshark.

This demonstrated how Expert Information can be used as an initial indicator of unusual protocol conditions.

---

# 18. HTTP Traffic Investigation

Another exercise involved navigating to:

```text
Packet 4
```

and applying the **Hypertext Transfer Protocol** field as a filter.

This demonstrated how Wireshark can automatically create display filters from packet fields.

I then investigated the displayed packet count.

Later, I navigated to:

```text
Packet 33790
```

and used **Follow HTTP Stream** to reconstruct the communication.

The server's HTTP response contained information about artists.

I analyzed the reconstructed response to determine:

- The total number of artists.
- The name of the second artist.

This was an important demonstration of why stream reconstruction is useful: the information of interest was easier to understand at the application level than by inspecting individual packets.

---

# Key Lessons

The most important lessons I took from this room are:

### 1. Packets are structured

A packet is not just a block of random data.

It contains multiple protocol layers:

```text
Frame
 ↓
Ethernet
 ↓
IP
 ↓
TCP/UDP
 ↓
Application Protocol
 ↓
Application Data
```

---

### 2. The question determines where you look

For example:

```text
TTL              → IP
TCP payload      → TCP
HTTP E-Tag       → HTTP
Arrival time     → Frame
```

This makes packet analysis much more systematic.

---

### 3. Filtering is essential

Large packet captures can contain enormous amounts of traffic.

Display filters allow analysts to reduce the noise and focus on relevant traffic.

---

### 4. Following streams reveals the conversation

Individual packets provide pieces of communication.

Following a stream reconstructs those pieces into a more understandable conversation.

---

### 5. Wireshark is an analysis tool, not an IDS

Wireshark does not automatically determine whether traffic is malicious.

It provides the packet-level evidence.

The analyst must understand the protocols, recognize anomalies and investigate the evidence.

---

# Conclusion

Completing **Wireshark: The Basics** gave me my first practical experience with packet analysis.

Before starting the room, Wireshark looked complicated because of the large amount of information displayed at once. After working through the exercises, I learned how to break that information down into manageable layers and investigate it systematically.

The most important workflow I learned was:

```text
Capture
   ↓
Navigate
   ↓
Identify protocol
   ↓
Inspect packet
   ↓
Filter traffic
   ↓
Follow conversation
   ↓
Extract information
   ↓
Investigate evidence
```

This room provided the foundation for deeper Wireshark analysis and network-forensics work.

## Next Step

The next room in my learning path is:

**Wireshark: Packet Operations**

The goal is to build on these fundamentals and investigate packets in greater depth.