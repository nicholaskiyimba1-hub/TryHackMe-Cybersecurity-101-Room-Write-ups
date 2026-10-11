# TryHackMe Blue

## Introduction

In this room, I am learning how to investigate a Windows target and understand how the EternalBlue vulnerability can be identified during reconnaissance. EternalBlue is associated with a vulnerability in Microsoft's implementation of the Server Message Block version 1 protocol. The Blue room uses this vulnerability as a practical learning exercise.

The first stage is reconnaissance. Before attempting to exploit a target, I need to gather information about the services it exposes and identify which services may require further investigation. In this room, I use Nmap to scan the target machine and inspect the services and versions that it reports.

All scanning and exploitation activities described in these notes are intended for the authorized TryHackMe lab environment.

## Reconnaissance with Nmap

Nmap is a network scanning tool that can identify hosts, discover open ports, and gather information about the services listening on those ports. Its capabilities depend on the scan options I use and on what the target allows the scan to discover.

I start by obtaining the target machine's IP address from the TryHackMe room. I then run an Nmap scan with service-version detection enabled.

```bash
nmap -sV <target_ip>
```

The `-sV` option asks Nmap to probe discovered open ports to help identify the services and their versions. This information can help me decide what to investigate next.

The scan output normally includes the port number, state, service name, and any version information Nmap can identify. An open port indicates that a service is accepting connections on that port. A service name or version reported by Nmap is an identification result, not by itself proof that the service is vulnerable.

## Counting Open Ports Below 1000

One of the room's reconnaissance questions asks how many open ports have port numbers below 1000.

To answer this, I read the Nmap results and count only the ports whose state is `open` and whose port number is less than 1000. I do not count closed or filtered ports, and I do not include port 1000 because the question specifies port numbers below 1000.

In the walkthrough material I studied, three open ports were shown below port 1000. I should verify that result against the scan output from my own active target because lab results can depend on the target instance and its current state.

## Identifying a Service for Further Investigation

Nmap results help me decide which services deserve closer examination. I should not assume that a service is vulnerable merely because it uses a particular port. Instead, I compare the exposed services and their reported versions with reliable information about known vulnerabilities.

In this room, SMB on TCP port 445 is especially relevant. SMB is a protocol used by Windows systems for functions such as file and printer sharing. Port 445 is commonly associated with direct-hosted SMB traffic.

The service and version information reported for port 445 provides a reason to investigate the target's SMB implementation further. However, a scan result alone does not confirm that EternalBlue is exploitable. I need to consider the target's operating system, SMB version, patch status, and other evidence.

## Understanding EternalBlue

EternalBlue is the name commonly associated with exploitation of the Windows SMBv1 vulnerability tracked as CVE-2017-0144. The vulnerability was addressed by Microsoft in a security update released in March 2017. It became widely known in 2017 and was later used in major attacks, including WannaCry.

The Blue room uses EternalBlue as the central vulnerability for the lab. Understanding the relationship between SMB, the target's Windows version, and the vulnerability is more useful than memorizing a port number alone.

Port 445 is a useful reconnaissance clue because it commonly exposes SMB, but SMB being open does not automatically mean the target is vulnerable to EternalBlue. A modern, patched Windows system can have SMB enabled without being vulnerable to this specific flaw.

## How I Approach the Reconnaissance Question

When I am asked what a target may be vulnerable to based on an Nmap scan, I begin by reviewing the service names and version information. I look for services or versions that stand out and then research the relevant technology and known vulnerabilities.

I can use reputable vulnerability advisories, vendor security bulletins, and established exploit references to investigate a finding. The Exploit Database can be useful for learning about publicly documented exploits, but the existence of an exploit entry does not prove that a particular target is vulnerable.

In this room, SMB on port 445 is the main lead because the exercise focuses on EternalBlue. I should treat this as a hypothesis to investigate rather than a conclusion based only on the port number.

## What I Learned

This reconnaissance stage taught me that Nmap is useful for building an initial picture of a target's exposed services. The `-sV` option adds service-version information that can guide further investigation.

I also learned to read scan results carefully when counting ports. I must count only ports marked `open` and apply the exact boundary specified in the question.

The most important lesson is that reconnaissance produces leads, not automatic proof of vulnerability. In this room, SMB on port 445 directs my attention toward EternalBlue, but confirming a vulnerability requires more than noticing that the port is open.

## Commands to Remember

The main command from this section is:

```bash
nmap -sV <target_ip>
```

I use it to scan the target and ask Nmap to identify services and their versions where possible. I then interpret the output, identify services that warrant investigation, and compare the findings with reliable vulnerability information before drawing conclusions.
