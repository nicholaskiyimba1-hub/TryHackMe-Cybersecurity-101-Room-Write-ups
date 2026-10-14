# Firewall Fundamentals: Firewall Types, Rules, Windows Defender Firewall and UFW

## Introduction

This writeup documents what I learned in the TryHackMe Firewall Fundamentals room. The room explains what a firewall does, compares the main types of firewalls, describes the parts of a firewall rule, and then gives two guided practical exercises. In the first I created and inspected rules in Windows Defender Firewall, and in the second I used UFW on the Linux AttackBox to enable a firewall and manage simple rules. I wrote this as a reference for myself, so I have tried to explain the ideas and the commands and not only the steps I followed in the room. These were guided learning exercises, and I have not configured firewalls in a real production network.

## What a firewall does

A firewall is a security control that inspects the traffic going into and out of a device or a network and decides whether each piece of traffic is allowed. The easiest way for me to think about it is as a filter placed in the path of communication. Anything I send to the internet, or receive from it, can pass through a firewall that compares the traffic against a set of rules.

Firewalls are not only separate security appliances. Most computers have a firewall built into the operating system, and a home router usually has one as well. This means that when I visit a website, my traffic can pass through several firewalls, including the one on my computer, the one on my router, and the ones protecting the network of the website. Having rules at each point allows fine control over what is permitted.

To understand how the types of firewalls differ, I found it useful to know the OSI model. In this room three layers mattered. Layer 3 is the network layer, where IP addresses are used. Layer 4 is the transport layer, where protocols such as TCP and UDP and their port numbers are used. Layer 7 is the application layer, where the actual content of the communication, such as a web request, can be seen. The type of firewall largely depends on how much of this information it can examine.

## Types of firewalls

### Stateless firewalls

A stateless firewall works at layers 3 and 4 and makes its decision by comparing each packet against a list of rules. The rules can be based on the source or destination IP address, the port number, and the protocol, or any combination of these. If the packet matches a rule that allows it, it passes, and if it matches a rule that blocks it, it is dropped. This is the earliest and simplest style of firewall, and it is fast and easy to understand. The word stateless means that the firewall keeps no memory of earlier packets. Every packet is judged on its own, so the firewall does not know whether a packet is part of an existing connection or whether the same source has already sent many blocked packets.

### Stateful firewalls

A stateful firewall also works at layers 3 and 4 and uses the same kind of rules, but it additionally tracks the state of connections. It keeps a table of active connections, so it can tell whether an incoming packet belongs to a connection that was already established, for example a reply to a request that my own computer made. This allows a firewall to accept responses to traffic I started while rejecting unsolicited traffic from outside.

In my notes I described a stateful firewall as something that would remember an attacker after several failed attempts and block that address permanently. That description is not accurate for a basic stateful firewall. Tracking connection state is what makes it stateful, while blocking a source after repeated failures is usually done by additional features such as rate limiting, an intrusion prevention system, or a separate tool. The general idea I took from the room, that a stateful firewall keeps history and connection context while a stateless one does not, is still correct.

### Proxy firewalls

A proxy firewall sits between the client and the destination. My computer sends its request to the proxy, the proxy sends the request to the internet on my behalf, and the response comes back to the proxy before it is passed to me. Because it handles the full communication, it can inspect the content of the traffic and not only the addresses and ports. This is why it operates at layer 7, the application layer. A second effect is that the destination sees the address of the proxy and not my own address, so the proxy also hides the internal client.

### Next generation firewalls

A next generation firewall works from layer 3 up to layer 7 and combines the filtering of stateful firewalls with deep packet inspection, which means it examines the content of the traffic. It also provides additional security features. The room highlighted heuristic analysis, which is detection based on suspicious behavior and patterns and not only on exact matches with known bad traffic. I also understood that these firewalls can be connected to threat intelligence feeds, which are continuously updated lists of known malicious IP addresses and domains, so the firewall can block them automatically.

I was confused about the difference between a proxy firewall and a next generation firewall, since both can inspect content. My current understanding is that a proxy firewall is defined by acting as an intermediary that hides the client, while a next generation firewall is defined by combining many inspection and detection features, and it does not need to act as a proxy. I formed this understanding partly by looking the terms up, so I treat it as a working explanation and not as expert knowledge.

On the word heuristic, I learned that it describes a quick, experience-based way of judging something without following only a fixed rule. In a firewall it means looking at patterns of behavior and deciding whether they look suspicious, which allows detection of threats that do not match any known signature, although it can also produce false positives.

## Firewall rules

A firewall rule is made of a few basic parts. The source address says where the traffic comes from, the destination address says where it is going, the port and the protocol describe the kind of service, the direction says whether the traffic is entering or leaving the network, and the action says what the firewall should do with the matching traffic. Inbound means traffic coming into my network or device, and outbound means traffic leaving it.

The room described three actions. Allow permits the traffic. Deny blocks the traffic by dropping it. Forward sends the traffic on to a different destination. As an example from the room, a rule can allow outbound TCP traffic on port 80 from any source to any destination, which would permit web browsing over HTTP. A different example rule denies inbound TCP traffic on port 22 from any source, which blocks SSH connections coming into the network. A forward rule could take incoming traffic on port 80 and send it to a specific web server inside the network, for example one at `192.168.1.8`. These are the room's example rules and not rules from a real system.

## Windows Defender Firewall

Windows has a built-in firewall called Windows Defender Firewall. It can be found by searching for firewall in the Start menu. The main page shows whether the firewall is on for each network profile, and it offers options to allow an app through the firewall, to restore the default settings, and to open the advanced settings.

Network profiles are sets of rules that apply depending on the type of network the computer is connected to. The private profile applies to a trusted network such as a home network, the public profile applies to networks like a coffee shop where other people share the network, and the domain profile applies when the computer is joined to an Active Directory domain, as is common in a company. Windows can detect which type of network it is on and apply the matching profile.

The advanced console is where individual inbound and outbound rules are created and reviewed. In my notes I wrote that opening the advanced security entry from search leads to the wrong menu, but I am not sure about that. The console named Windows Defender Firewall with Advanced Security is the one that lists the inbound and outbound rules, and the Advanced settings link from the main page opens that same console.

### Creating a custom outbound rule

In the room I created an outbound rule that blocked web traffic. I opened the outbound rules section and chose to create a new rule of the custom type, which allowed me to define each part. I applied the rule to all programs, selected TCP as the protocol, and set the remote ports to the specific ports `80,443`, written without a space after the comma. Port 80 is used for HTTP and port 443 for HTTPS, so together these cover normal web browsing. I left the scope as any IP address, chose the action to block the connection, applied the rule to all profiles, and gave it a name.

I should be honest that I was not able to demonstrate the before and after effect. The test page from the room did not load for me even before I created the rule, so I could not confirm the rule changed anything in that case. I understood the principle, which is that a rule blocking outbound TCP ports 80 and 443 would stop normal web browsing from that computer. A rule like this would not be sensible on a normal everyday computer, but it can make sense on a machine that is dedicated to a task that should not use the internet.

### Reading existing rules

For the second part, I reviewed rules that had already been created. I learned that I must first make sure I am looking at the correct rule set. I began in outbound rules, but the questions asked about incoming traffic, so I needed the inbound rules. In the inbound rules I found a rule that blocked all incoming traffic on TCP port 22, which is the SSH port, and another rule that allowed SSH on port 22 only from a single remote IP address. Reading the local port, remote address, protocol, and action columns was enough to tell what each rule did. The list can also be filtered, for example by port number or by whether the rule allows or blocks, which is faster than scrolling when many rules exist.

## Linux firewalls

Linux has firewall functionality built into the kernel through a framework called Netfilter. Netfilter itself does the filtering, and the tools I would use are programs that give me a way to configure it. The most common one is `iptables`, which I learned is used widely but has syntax that takes practice. The tool `nftables` is its successor, with more capabilities. The tool `firewalld` is another option, and it works with predefined zones that describe different levels of trust for network connections. UFW, which stands for Uncomplicated Firewall, is a simpler interface designed to make common rules easy to write for beginners.

### Using UFW

On the AttackBox I was already working as the root user, so I did not need `sudo`. On a normal Linux system, these commands would need it. I first checked the state of the firewall.

```bash
ufw status
```

It reported that the firewall was inactive, so I turned it on.

```bash
ufw enable
```

I then set the default policy for outgoing traffic. The word default means that this policy applies to any outgoing traffic that is not covered by a more specific rule. Allowing outgoing traffic by default means that my machine can start connections unless I block something specifically.

```bash
ufw default allow outgoing
```

Next I added a rule to deny incoming TCP traffic on port 22, which blocks SSH connections to the machine. UFW answered that the rule had been added. In my notes I mistyped this command a few times, with a backslash and with a wrong name, and the correct form uses the port number and the protocol separated by a forward slash.

```bash
ufw deny 22/tcp
```

To see the rules with their numbers I used the numbered status.

```bash
ufw status numbered
```

I then removed a rule by its number, which asks for confirmation before it deletes anything. After the deletion, the numbered list showed only one remaining rule.

```bash
ufw delete 2
```

The last question asked how to deny all outgoing traffic as the default policy. By following the same structure as the allow command, the answer is to change the word allow to deny. I understood this as a general pattern, where the default command takes a policy, which is allow or deny, and a direction, which is incoming or outgoing.

```bash
ufw default deny outgoing
```

As general knowledge that was not part of my practice, denying all outgoing traffic by default is a strong setting. It would block things such as DNS lookups and updates unless I add rules that allow them. Also, on a remote machine, enabling a firewall that blocks port 22 can cut off my own SSH session, so the rules need to be planned before they are applied.

## What I learned

I learned that a firewall is a rule-based filter and that the types differ in how much they can see and remember. Stateless firewalls compare each packet with rules at layers 3 and 4, stateful firewalls also track connections, proxy firewalls sit in the middle and inspect content at layer 7, and next generation firewalls combine deep inspection with features such as heuristic analysis and threat intelligence. I learned the parts of a firewall rule, which are source, destination, port, protocol, direction, and action, and how the allow, deny, and forward actions work. In Windows, I learned how profiles work, how to create a custom outbound rule, and how to read inbound rules while paying attention to direction. On Linux, I learned that Netfilter is the underlying framework, that several tools configure it, and that UFW offers a simple way to enable a firewall, add and delete rules, and set default policies.

## Conclusion

This room gave me the basic vocabulary and a first practical experience with host firewalls on Windows and Linux. I can now explain the main firewall types, describe what a rule contains, create a simple rule in Windows Defender Firewall, and manage basic rules with UFW. My experience is limited to these guided exercises. I have not used `iptables` or `nftables` directly, I have not worked with a proxy or next generation firewall, and I want to study the difference between those two types further.
