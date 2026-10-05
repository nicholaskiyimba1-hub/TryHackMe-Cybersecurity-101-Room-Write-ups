# Metasploit Introduction (TryHackMe)

## Introduction

This writeup documents what I learned and practiced while working through the Metasploit Introduction room on TryHackMe. The room covers what Metasploit is, how it is organized, and how to use the msfconsole interface to find, configure, and run an exploit module. I am writing this as a personal technical record so that I can come back to it later and still understand how the framework works and why the commands do what they do.

## What Metasploit Is

Metasploit is a penetration testing framework that is used across many stages of a security assessment, including information gathering, scanning, exploitation, and post exploitation. I learned that it is built around three main parts. The msfconsole is the command line interface I use to interact with the framework, and it is started with the command `msfconsole`. Modules are the scripts and components that carry out specific tasks such as scanning a target or delivering an exploit. Exploits are the actual pieces of code that take advantage of a vulnerability on the target system. Understanding Metasploit as these three connected parts made it easier to follow the rest of the room, because almost every command I used was either about finding a module, inspecting a module, or running one.

## Core Security Concepts

Before working with the commands, the room introduced a few terms that are worth explaining properly because they are easy to confuse with each other. A vulnerability is a weakness in a system, which can come from a logic flaw or a coding mistake, and it is the thing that gives an attacker a way to compromise the confidentiality, integrity, or availability of the target. An exploit is the actual code that takes advantage of a specific vulnerability in order to carry out an attack against the target. A payload is different again: it is the code that actually runs on the target once the exploit has succeeded, and it is what lets the attacker achieve whatever goal they have on that system, whether that is running a command, opening a shell, or something else. Keeping these three terms separate helped me understand that an exploit is the mechanism that gets you in, while the payload is what you actually do once you are in.

## Module Categories

Metasploit organizes its modules into different categories depending on what job they do, and the room walked through each of these. Auxiliary modules are supporting modules that are not exploits themselves, and they are generally used for tasks like scanning or gathering information rather than directly compromising a target. Encoders are used to disguise or obfuscate a payload so that its actual content is harder to identify, which is relevant when trying to avoid detection. Evasion modules serve a similar purpose but are focused specifically on helping a payload avoid being picked up by antivirus software running on the target. Post modules are used after a system has already been exploited, for example to escalate privileges or gather further information from a system the attacker already has access to.

The room also covered stagers, stages, and singles, which describe different ways a payload can be delivered to a target. A stager is a small piece of code whose only job is to establish the initial connection between the attacker machine and the target. Once that connection exists, a stage can be downloaded over it, which is how a larger or more capable payload gets onto the target without needing to be part of the original exploit. A single, by contrast, is a self contained payload that does not need to download anything extra because it is small enough to run on its own. I also came across NOP modules in the room. NOP stands for no operation, and in the context of memory based exploits these are used as padding so that a payload lines up correctly in memory, which helps the exploit run more reliably. I related this back to what I already knew about operating systems and how a scheduler allocates processing time to different processes, since the general idea of using space or time to let something work smoothly without interruption felt similar, even though NOPs are a much more specific, low level technique.

## The MSF Console and Its Commands

The msfconsole is the interface the room used for everything, and this section covers the commands I actually practiced inside it. Some of these commands behave the same way they would on a normal Kali Linux terminal, which I found interesting, since msfconsole still accepts something like `cd` to change directory even though it is its own environment.

The `help` command is used to see the available commands and how to use them, which is a reasonable first step if I forget the syntax for something. The `search` command is what I used to look for modules related to a particular topic. For example, running

```bash
search ms17
```

returned a list of modules related to the MS17 vulnerability, including both exploit modules and auxiliary modules, along with information such as the disclosure date and a rank describing how reliable the module is. The ranks range from low up to excellent, and the room made the point that a high rank does not guarantee the exploit will work cleanly every time, since even an excellent ranked module can behave unexpectedly or fail against a particular target. Some modules also support a `check` function, which lets me test whether a target appears vulnerable before actually running the exploit against it.

Search results can also be narrowed down using filters, so instead of searching for a loose keyword I can combine the module type and the target platform with the keyword itself, for example

```bash
search type:auxiliary platform:linux telnet
```

which restricts the results to auxiliary modules that target Linux systems and relate to Telnet. This turned out to be a much more practical way of searching once the list of possible modules got long.

Once I find a module I want to use, I load it with the `use` command followed by either its number from the search results or its full module path. Using the number from the list is faster, since typing the entire module path is not necessary. After loading a module, `show options` lists the options the module needs, and it distinguishes between options that are already filled in by default and ones that still need a value before the module can be run. If I want to leave the module and return to the main msfconsole prompt, the `back` command does that.

The `info` command is related to `show options` but serves a different purpose. While `show options` tells me what I need to configure to run the module, `info` describes the module itself: it gives a description, lists the target types it supports, shows some of its options, and names who provided the module. I practiced this directly while working through one of the room's questions, which asked who provided a particular auxiliary SSH login scanning module. After loading that module with `use`, running `info` showed the author's name in the module description, which answered the question.

Setting values for a module's options is done with the `set` command, followed by the option name and the value I want to assign. The option names follow a fairly consistent pattern across modules. `RHOSTS` is the remote host, meaning the target I am pointing the module at, and `RPORT` is the remote port. `LHOST` and `LPORT` are the listening host and port used when the exploit needs to set up a connection back to the attacker machine, and `PAYLOAD` sets which payload the module will deliver if it succeeds. If I need to clear a value I have set, `unset` followed by the option name removes it, and `unset all` resets every option in the current module back to its defaults rather than clearing them one at a time.

One detail that caused me some confusion while working through this part of the room was the difference between a normal `set` and a global `set`. Running `set g` followed by an option name and value sets that option globally, which means the value stays set across every module for the rest of the msfconsole session rather than only inside the module I am currently in. I tested this by setting `RHOSTS` globally, then switching to a different MS17 related module, and confirming that the same IP address was already filled in without me setting it again. I then used `unset g` to clear the global value and confirmed that both modules were empty again. It is worth remembering that a global value only lasts for the current msfconsole session and resets once the console is closed and reopened.

Once a module's options are configured, running it is done with either `exploit` or `run`, which do the same thing. I used `run` mostly because it is shorter to type. After a module runs successfully against a target, the `sessions` command lists any active sessions that resulted from it, and `sessions` followed by a session number lets me interact with a specific one if there is more than one open. If I am inside a session and want to return to the main msfconsole prompt without closing the session, the `background` command, or pressing Ctrl and Z, sends the session to the background so it stays available to come back to later.

## Practical Work

The practical part of the room had me go through the full workflow described above against a provided target. I searched for the MS17 related exploit using `search ms17`, loaded the correct module from the results with `use` and its number, checked what it needed with `show options`, and set the target address with `set RHOSTS` pointed at the provided IP. With the required option filled in, I ran the module using `run`.

This resulted in a Meterpreter session on the target. Meterpreter is a payload that is specific to Metasploit and provides more functionality for post exploitation work than a plain shell would. Once inside the session I ran basic commands such as checking the working directory and listing files, and these behaved in a Linux like way even though the target system being exploited was Windows. The reason for this, as I understood it from the room, is that Meterpreter provides its own consistent command set that works the same way regardless of what operating system the target is actually running.

I also practiced moving a session to the background with the `background` command and then returning to it using `sessions` followed by the session number, which let me confirm I was still inside the same session on the target's filesystem as before.

## What the Practical Work Demonstrated

Working through this exercise showed me that msfconsole is really a small number of repeated steps rather than a large number of separate commands to memorize. Almost everything comes down to searching for a module, loading it, checking and setting its options, and then running it, with `info` and `show options` both playing a role in understanding what a module needs before it is run. The exercise also showed me why a successful exploit is not the end of the process, since the resulting Meterpreter session is itself something that needs to be managed, interacted with, and backgrounded when needed.

## My Takeaways

Going through this room helped me understand the msfconsole workflow as a connected process rather than a list of disconnected commands, which is something the room's task by task structure did not make obvious on its own. I initially mixed up `show options` and `info`, but working through both directly made the difference clear, since `show options` is about what still needs to be filled in while `info` is about understanding the module itself. The distinction between stagers, stages, and singles also took me a couple of attempts to get straight, and relating the padding function of NOPs back to general operating system concepts helped it make more sense to me. This was a guided introductory lab rather than independent testing, and I want to be clear about that, but it gave me a working sense of how Metasploit is structured and how to move through a module from search to exploitation.
