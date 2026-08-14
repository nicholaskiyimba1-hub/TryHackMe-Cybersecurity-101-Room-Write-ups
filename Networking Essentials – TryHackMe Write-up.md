# Networking Essentials – TryHackMe Write-up

## Room Overview

In this room, I learned the essential networking protocols and services that allow devices to communicate on local networks and across the Internet. The room focused on understanding **how a device joins a network, communicates with other devices, accesses the Internet, and troubleshoots connectivity issues**.

The topics covered include:

- DHCP (Dynamic Host Configuration Protocol)
- ARP (Address Resolution Protocol)
- ICMP (Internet Control Message Protocol)
- Routing
- NAT (Network Address Translation)

---

# Learning Objectives

After completing this room, I can:

- Explain how devices automatically obtain network configurations.
- Describe how IP addresses are translated into MAC addresses.
- Use ICMP for connectivity testing and troubleshooting.
- Understand how routers determine paths across networks.
- Explain why NAT is necessary and how it conserves IPv4 addresses.
- Interpret packet captures related to these networking protocols.

---

# DHCP (Dynamic Host Configuration Protocol)

## Purpose

DHCP automatically assigns network configurations to devices when they join a network.

Without DHCP, every device would require manual configuration of:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

This would become impractical in large networks.

---

## The DORA Process

DHCP operates using four steps known as **DORA**.

### 1. Discover

The client broadcasts a DHCP Discover packet to locate a DHCP server.

- Source IP: **0.0.0.0**
- Destination IP: **255.255.255.255**

The client has not yet received an IP address, so it must broadcast.

---

### 2. Offer

The DHCP server responds with an available IP address and other network configurations.

---

### 3. Request

The client accepts the offered IP address by sending a DHCP Request.

---

### 4. Acknowledge

The DHCP server confirms the lease by sending a DHCP ACK.

The client is now fully configured and can communicate on the network.

---

## Key Concepts

- DHCP uses UDP.
- Server Port: **67**
- Client Port: **68**
- Clients begin with IP address **0.0.0.0**.
- Broadcast address used: **255.255.255.255**.

---

# ARP (Address Resolution Protocol)

## Purpose

Devices communicate using IP addresses at Layer 3, but Ethernet requires MAC addresses at Layer 2.

ARP translates:

**IP Address → MAC Address**

---

## Why ARP Exists

Suppose a device wants to communicate with:

192.168.66.1

The sender knows the IP address but not the MAC address.

ARP solves this problem.

---

## ARP Request

The sender broadcasts:

> Who has 192.168.66.1?

Destination MAC:

```
ff:ff:ff:ff:ff:ff
```

This is the Ethernet broadcast address.

---

## ARP Reply

The target device replies with:

> 192.168.66.1 is at 44:df:65:d8:fe:6c

The sender stores this information in its ARP cache and can now communicate directly.

---

## Important Notes

- ARP operates directly over Ethernet.
- It is not encapsulated inside IP or UDP.
- ARP bridges Layer 3 addressing with Layer 2 addressing.

---

# ICMP (Internet Control Message Protocol)

## Purpose

ICMP is used primarily for:

- Diagnostics
- Connectivity testing
- Error reporting

It is not used for carrying application data.

---

## Ping

Ping checks whether another device is reachable.

It sends:

ICMP Echo Request (Type 8)

The destination replies with:

ICMP Echo Reply (Type 0)

Ping also measures the Round Trip Time (RTT).

---

## Traceroute

Traceroute discovers the path packets take across routers.

It relies on the IP header field:

**Time To Live (TTL)**

Each router decreases TTL by one.

When TTL reaches zero:

- The router discards the packet.
- It sends back an ICMP Time Exceeded (Type 11) message.

By gradually increasing TTL values, traceroute discovers every router along the path.

---

## Key Concepts

- Ping determines whether a host is reachable.
- Traceroute reveals intermediate routers.
- RTT measures latency.
- Packet loss may indicate congestion or connectivity issues.

---

# Routing

## Purpose

Routing determines the best path for packets traveling between different networks.

Routers forward packets toward their destination using routing tables.

Each router only decides the next hop.

---

## Routing Protocols Covered

### OSPF

- Open standard
- Shares network topology
- Calculates the shortest path

---

### EIGRP

- Cisco proprietary protocol
- Uses bandwidth and delay metrics
- Fast convergence

---

### BGP

- Primary routing protocol of the Internet
- Exchanges routes between different organizations (Autonomous Systems)

---

### RIP

- Simple routing protocol
- Chooses the route with the fewest hops

---

## Comparison

| Protocol | Purpose |
|----------|----------|
| OSPF | Shortest path routing |
| EIGRP | Cisco proprietary routing |
| BGP | Internet-wide routing |
| RIP | Hop-count based routing |

---

# NAT (Network Address Translation)

## Purpose

IPv4 provides approximately 4.3 billion addresses.

Due to the rapid growth of Internet-connected devices, public IPv4 addresses became scarce.

NAT allows multiple private devices to share one public IP address.

---

## How NAT Works

Internal devices use private addresses such as:

```
192.168.x.x
10.x.x.x
172.16.x.x
```

The router owns a public IP address.

When a device sends traffic to the Internet:

Private IP

↓

Router performs NAT

↓

Public IP

The Internet only sees the router's public address.

---

## NAT Translation Table

The router maintains a table that maps:

- Private IP
- Private Port

to

- Public IP
- Public Port

This allows thousands of simultaneous connections while using a single public IP.

---

## Benefits

- Conserves public IPv4 addresses.
- Allows many devices to share one Internet connection.
- Hides internal network addressing from external networks.

---

# Practical Exercise

The room included a practical Bash scripting exercise that searched log files for a flag.

Initially, the script contained empty quotation marks instead of the directory variable.

Original code:

```bash
for file in " "/*.log
```

After replacing the empty quotes with the correct variable:

```bash
for file in "$directory"/*.log
```

the script successfully searched every log file inside:

```
/var/log
```

The output identified the correct file:

```
authentication.log
```

I then displayed its contents using:

```bash
cat /var/log/authentication.log
```

Inside the file, I found:

```
the cat is sleeping under the table
```

along with the room flag.

---

## What I Learned From the Practical

This exercise reinforced several important Linux concepts:

- Variables must be referenced correctly using `$variable`.
- Bash expands wildcard patterns such as `*.log`.
- File paths must be written correctly.
- Absolute paths prevent file lookup errors.
- Reading error messages helps locate mistakes quickly.

---

# Key Takeaways

After completing this room, I understand how the major networking components work together:

1. **DHCP** automatically assigns network configuration.
2. **ARP** translates IP addresses into MAC addresses.
3. **ICMP** diagnoses network connectivity.
4. **Routing** forwards packets across different networks.
5. **NAT** enables multiple devices to share a public IP address.

These technologies work together whenever a device connects to a network and accesses resources on the Internet.

---

# Skills Gained

- Networking Fundamentals
- DHCP Configuration Process
- ARP Resolution
- ICMP Diagnostics
- Ping and Traceroute
- Routing Concepts
- Routing Protocol Basics
- NAT Fundamentals
- Packet Flow Analysis
- Linux Networking Concepts

---

# Final Thoughts

This room strengthened my understanding of how devices communicate across local and global networks. Rather than viewing protocols such as DHCP, ARP, ICMP, Routing, and NAT as isolated concepts, I now understand how they work together to enable seamless communication—from obtaining an IP address to successfully reaching a destination on the Internet. The practical exercise also reinforced Bash scripting fundamentals and highlighted the importance of careful troubleshooting when working with Linux systems.