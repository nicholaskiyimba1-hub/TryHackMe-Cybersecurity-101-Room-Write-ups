# Meterpreter

## Introduction

Meterpreter is a payload and command interpreter that is part of the Metasploit Framework. I can think of Meterpreter as Metasploit's specialized post-exploitation shell. After a successful exploitation or payload execution, I can receive a Meterpreter session on the target and use its built-in commands to interact with the compromised system.

A normal command shell mainly gives me access to commands provided by the operating system. Meterpreter provides its own command set and additional functionality designed for post-exploitation. This includes interacting with files, processes, network information, system information, privilege escalation features, credential collection, and other post-exploitation activities.

Meterpreter is delivered as a payload. The general idea is that an exploit or another delivery method gives me code execution on the target, and the payload establishes the Meterpreter session.

One important characteristic of Meterpreter is that it is designed to operate primarily from memory rather than being installed as a normal program on the target's disk. This can make some traditional file-based detection methods less effective. However, this does not mean Meterpreter is invisible or undetectable. Modern endpoint security products can detect Meterpreter through behavioral analysis, memory inspection, network monitoring, signatures, and other techniques.

Meterpreter can also operate within the context of another process. This is why understanding processes and process IDs becomes important when working with Meterpreter.

## Meterpreter Process Identification

The `getpid` command tells me the process ID of the process in which my Meterpreter session is currently running.

```text
getpid
```

The process ID, or PID, allows me to identify the process associated with my current Meterpreter session.

The `ps` command displays the processes currently running on the target system.

```text
ps
```

I can use the PID returned by `getpid` and find the same PID in the output of `ps`. The process name may be something that looks normal for the operating system rather than something obviously named Meterpreter.

For example, a Meterpreter session could be associated with a process such as `spoolsv.exe`. The important point is that the process name itself does not necessarily reveal that Meterpreter is running.

Meterpreter can also make use of process injection and migration techniques. These techniques allow the Meterpreter payload to move its execution context into another process.

## Meterpreter and DLLs

Windows processes use Dynamic Link Libraries, commonly called DLLs, to provide reusable code and functionality. Meterpreter can operate in a process context where its components are loaded into memory rather than existing as an obvious file named something like `meterpreter.dll`.

This contributes to the difficulty of detecting some Meterpreter activity using simple file inspection alone. However, modern security software does not rely only on looking for a specific file name.

## Meterpreter Network Communication

Meterpreter supports different communication methods depending on the payload that is being used. Examples include TCP, HTTP, and HTTPS-based payloads.

An HTTPS Meterpreter payload can encrypt its network communication. This means an organization that can only observe encrypted traffic without decrypting or inspecting it may have less visibility into the actual contents of the communication.

Encryption does not make the connection invisible. Network monitoring can still identify suspicious connections through metadata, traffic patterns, destination information, and other indicators.

## Meterpreter Flavors

Meterpreter payloads can be divided into single payloads and staged payloads.

A single payload, also called an inline payload, contains the components required for the payload in one package. The payload is delivered as a single unit.

A staged payload separates the initial payload from the larger Meterpreter component. The first stage establishes the connection and then retrieves or loads the second stage.

The choice between these payload types depends on the situation and the capabilities of the target.

MSFvenom can be used to list available payloads.

```bash
msfvenom --list payloads
```

I can filter the results for Meterpreter payloads by using `grep`.

```bash
msfvenom --list payloads | grep meterpreter
```

This produces many Meterpreter payload options for different operating systems, architectures, and communication methods.

Examples include Meterpreter payloads for Windows, Linux, Android, Java, and other platforms. I may also see different communication methods such as reverse TCP, reverse HTTP, and reverse HTTPS.

The appropriate Meterpreter payload depends mainly on the target operating system, the target architecture and available components, and the network communication that is possible between the target and the attacking machine.

For example, I cannot simply select a Windows payload when my target is a Linux system. The payload has to be compatible with the target.

Some Metasploit exploits also have default payloads. When I select an exploit, Metasploit may automatically provide a payload that is appropriate for that exploit.

Inside `msfconsole`, I can use the following command to see the payloads available for the selected exploit.

```text
show payloads
```

When creating a payload manually with MSFvenom, I need to select the payload myself and make sure it matches the target.

## Meterpreter Commands

Once I have a Meterpreter session, I can use the `help` command to see the commands available to me.

```text
help
```

The commands are grouped into categories such as core commands, file system commands, networking commands, system commands, and other Meterpreter functionality.

This makes `help` one of the most useful commands when I am learning Meterpreter because I do not need to memorize every command.

I can use the help output to discover commands and understand what each command is designed to do.

## Post Exploitation with Meterpreter

Meterpreter becomes particularly useful after gaining access to a target because it provides commands for post-exploitation activities.

The `getuid` command tells me which user account the Meterpreter session is running under.

```text
getuid
```

For example, if I receive:

```text
Server username: NT AUTHORITY\SYSTEM
```

the session is running with the Windows SYSTEM account, which has very high privileges.

The `ps` command shows the processes running on the target.

```text
ps
```

This becomes important when I need to identify a process for migration or investigate what is currently running on the system.

## Process Migration

The `migrate` command allows Meterpreter to move its execution context into another process.

```text
migrate <PID>
```

For example:

```text
migrate 1276
```

The PID must come from the process list returned by `ps`.

Process migration can be useful for several reasons. It can change the security context or privileges available to the session, help maintain a more suitable process context, or support other post-exploitation techniques.

I need to be careful when migrating because moving into an unsuitable process can cause problems or result in losing privileges. A process may also terminate, which can cause the Meterpreter session to disappear.

This is why I should inspect the process list before attempting migration rather than choosing a PID randomly.

## Hashdump

The `hashdump` command attempts to retrieve password hashes from the Windows Security Account Manager, commonly called the SAM database.

```text
hashdump
```

The SAM contains information associated with local Windows accounts and their password hashes. Retrieving this information generally requires sufficiently high privileges.

The output can contain multiple colon-separated fields for each account. The relevant password hash can then potentially be subjected to offline password cracking.

The important distinction for me is that `hashdump` is a Windows-specific Meterpreter capability for obtaining password hash information. It is not simply reading a normal text file containing passwords.

## Searching for Files

The `search` command has different meanings depending on whether I am using the Metasploit console or the Meterpreter console.

Inside `msfconsole`, I can use `search` to find Metasploit modules.

Inside a Meterpreter session, I can use the `-f` option to search for files.

```text
search -f flag2.txt
```

The `-f` option tells Meterpreter that I want to search by file name.

For example:

```text
search -f secrets.txt
```

Meterpreter searches the target system and can return the path where the file was found.

This distinction is important because the same word, `search`, performs different tasks depending on which console I am currently using.

## Opening a System Shell

Meterpreter has its own command environment, but sometimes I need to execute commands provided directly by the operating system.

The `shell` command can open a normal system shell.

```text
shell
```

Once I enter the system shell, I can use operating system commands that are not necessarily available as native Meterpreter commands.

For example, on Windows I can use:

```cmd
net share
```

This displays the network shares configured on the Windows system.

I can leave the system shell with:

```cmd
exit
```

This returns me to the Meterpreter session.

## The Post Exploitation Challenge

The practical challenge starts by using a Metasploit module to simulate the initial compromise of the Windows target.

The important idea is that the challenge does not require me to perform the original vulnerability discovery from scratch. Instead, the provided credentials and Metasploit module allow me to establish the initial Meterpreter session.

I first select the appropriate Metasploit module and inspect its options.

```text
use <module>
show options
```

I then configure the required target and authentication information.

```text
set RHOSTS <target_ip>
set SMBUser <username>
set SMBPass <password>
```

Before running the module, I should always use:

```text
show options
```

This gives me an opportunity to verify that the required options have been configured correctly.

I can then launch the module with:

```text
run
```

or:

```text
exploit
```

Once the exploitation succeeds, I receive a Meterpreter session.

## Checking the Target System

The `sysinfo` command provides basic information about the compromised system.

```text
sysinfo
```

The output can include information such as the computer name, operating system, architecture, and Meterpreter type.

The `getuid` command tells me the account associated with my current session.

```text
getuid
```

In the challenge, the session runs as:

```text
NT AUTHORITY\SYSTEM
```

This gives the session a very high level of privilege.

## Enumerating Windows Shares

I can use the Meterpreter `shell` command to enter a Windows command shell.

```text
shell
```

From there I can enumerate Windows shares with:

```cmd
net share
```

The output contains standard Windows administrative shares as well as shares created for specific purposes.

In the challenge, the unusual share was:

```text
speedster
```

The `speedster` share was the share associated with the user.

I can leave the Windows shell with:

```cmd
exit
```

and return to Meterpreter.

Another approach is to use a Metasploit post-exploitation module designed to enumerate shares. This demonstrates an important characteristic of Metasploit: the same information can sometimes be obtained either through native operating system commands or through dedicated Metasploit modules.

## Migrating to LSASS

One of the challenge questions required the NTLM hash of the `J Chambers` user.

Attempting to use `hashdump` immediately caused the session to fail in the room environment. The provided hint indicated that I first needed to migrate into the `lsass.exe` process.

I first listed the processes:

```text
ps
```

I then looked for:

```text
lsass.exe
```

The PID shown for the process in the challenge was:

```text
760
```

I migrated into that process with:

```text
migrate 760
```

After migration, I could verify the user context again with:

```text
getuid
```

The session remained:

```text
NT AUTHORITY\SYSTEM
```

I could then run:

```text
hashdump
```

and retrieve the Windows password hash information.

This demonstrated why understanding process migration is important. The ability to execute a command is not always enough. Some Meterpreter functionality depends on the process context in which the session is operating.

## Understanding the NTLM Hash Output

The `hashdump` output contains information associated with Windows accounts and their password hashes.

For the challenge, I needed the NTLM hash associated with the `J Chambers` account.

The hash was then used for offline password recovery.

The important lesson for me is that a password hash is not the same thing as the cleartext password. A hash is the result of a one-way cryptographic operation, and recovering the original password generally involves guessing candidate passwords and comparing their resulting hashes.

## Recovering the Cleartext Password

For the TryHackMe challenge, the recovered password was:

```text
trustno1
```

The walkthrough demonstrated an online hash-cracking service, but in a real penetration test I would normally perform password cracking in an authorized environment using an appropriate offline tool and wordlist.

John the Ripper and Hashcat are examples of tools commonly used for offline password cracking.

The important concept is that obtaining password hashes is one stage of credential access, while recovering the original password is a separate process.

## Searching for secrets.txt

The challenge then asked me to locate a file called `secrets.txt`.

I used the Meterpreter file search command:

```text
search -f secrets.txt
```

The search returned the location of the file.

I could then use the path returned by Meterpreter to access the file.

This reinforced the usefulness of `search -f` during post-exploitation because I do not necessarily need to know where a file is stored before searching for it.

## Reading Files with Spaces in Their Paths

When a file path contains spaces, I need to make sure the command interpreter treats the complete path as a single argument.

For example:

```text
cat "C:\Some Directory\secret file.txt"
```

Without quotation marks, the command interpreter may interpret the spaces as separators between different arguments.

This is a simple command-line issue, but it is important because a correct file path can still fail if it is not formatted correctly.

## Finding real secret.txt

The challenge also contained a second file named:

```text
real secret.txt
```

I searched for it using:

```text
search -f "real secret.txt"
```

The returned result provided the complete path to the file.

Because the filename contains a space, I needed to use quotation marks when referencing the path.

I could then read the file with the appropriate file-reading command.

The secret revealed by the challenge was:

```text
The flash is the fastest man alive.
```

## Loading Additional Meterpreter Extensions

Meterpreter can be extended with additional functionality.

The `load` command can be used to load extensions.

```text
load <extension>
```

For example:

```text
load kiwi
```

The `kiwi` extension provides functionality associated with Mimikatz techniques.

After loading an extension, the `help` command can display additional commands made available by that extension.

This means Meterpreter is not limited to only the commands available when the session is first established. Additional functionality can be loaded when required.

## Privilege Escalation with getSystem

Meterpreter provides the `getsystem` command for attempting privilege escalation on Windows.

```text
getsystem
```

The command attempts known privilege escalation techniques supported by Meterpreter.

If successful, the session can become associated with the Windows SYSTEM account.

I need to understand that `getsystem` is an automated attempt rather than a guarantee that privilege escalation will always succeed. Its success depends on the target system, its configuration, available vulnerabilities, and the techniques supported by the Meterpreter version.

## Important Relationships I Learned

Meterpreter is part of the Metasploit ecosystem, but it is different from the Metasploit console itself.

The `msfconsole` prompt indicates that I am interacting with the Metasploit Framework.

The `meterpreter` prompt indicates that I am interacting with an established Meterpreter session.

This distinction is important because commands behave differently depending on which environment I am currently using.

For example, `search` in `msfconsole` is used to search for Metasploit modules.

```text
search <keyword>
```

Inside Meterpreter, I can use:

```text
search -f <filename>
```

to search the target system for a file.

The `ps` command allows me to inspect processes, `getpid` tells me the PID associated with my current Meterpreter process, and `migrate` allows me to move into another process.

The `getuid` command tells me which user context I currently have, while `getsystem` can attempt to escalate privileges on Windows.

The `hashdump` command can retrieve Windows password hash information when the session has sufficient privileges.

The `shell` command lets me leave the Meterpreter command environment temporarily and interact with the target through its normal operating system shell.

## What I Learned from the Challenge

The practical challenge showed me that Meterpreter is more than just a remote command shell. It provides a structured post-exploitation environment with commands for system enumeration, process management, file searching, credential access, privilege escalation, and interaction with the underlying operating system.

I also learned that having a high privilege such as `NT AUTHORITY\SYSTEM` does not automatically mean every Meterpreter function will work from the current process context. Process migration can sometimes be necessary for specific operations.

I learned that I need to pay attention to the difference between Metasploit commands and Meterpreter commands. I also learned that when a Meterpreter command does not provide the functionality I need, I can sometimes use the `shell` command and perform the operation using the operating system itself.

The challenge also reinforced the importance of checking my options before running a Metasploit module. A simple configuration mistake such as failing to set `RHOSTS` can prevent an otherwise correct module from working.

I also saw that lab environments can sometimes behave inconsistently. A session may fail or die even when the configuration appears correct. In that situation, I should first verify my configuration and then consider restarting the affected lab machine if the problem persists.

## My Main Takeaway

My main understanding of Meterpreter is that it is a Metasploit post-exploitation payload and command interpreter designed to give me extensive control and enumeration capabilities after gaining access to a target.

I now understand that Meterpreter can operate from memory, communicate using different payload transports, interact with processes, migrate between processes, search for files, open system shells, retrieve Windows password hashes, attempt privilege escalation, and load additional extensions.

The commands I consider particularly important from this room are:

```text
help
getuid
getpid
ps
migrate <PID>
hashdump
search -f <filename>
shell
sysinfo
getsystem
load <extension>
sessions
```

I also understand that Meterpreter functionality depends heavily on the target operating system, architecture, process context, privileges, payload type, and network connection.

This room helped me connect the concepts from the earlier Metasploit exploitation material with actual post-exploitation activity. Instead of only using Metasploit to exploit a vulnerability and obtain a session, I can now understand what I can do after the session has been established and why the Meterpreter environment is useful during that stage of an authorized penetration test.
