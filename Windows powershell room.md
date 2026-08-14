# Windows PowerShell

**Write-up by: Nicholas Kiyimba**

## Introduction

This is the second room in the Command Line module, following on from Windows Command Line. Where that room focused on `cmd.exe`, this one introduces PowerShell — what it is, its object-oriented approach, its basic language structure, and practical use of its cmdlets for navigating the filesystem, filtering data, and pulling system/network information.

PowerShell matters a lot in cybersecurity specifically, not just as an admin convenience. It's deeply integrated into Windows, capable of very powerful system interaction, and because of that it's also one of the most commonly abused tools by attackers for post-exploitation activity (running scripts, moving laterally, pulling system data) — which is exactly why defenders and pentesters alike need to understand it well.

## Task 1 — Introduction

### Explanation

This task laid out the room's learning objectives: understanding what PowerShell is and what it can do, its basic language structure, running basic commands, and its relevance to cybersecurity work. It recommended having gone through the Windows and Active Directory Fundamentals module and the Windows Command Line room first, both of which I'd already completed.

## Task 2 — What is PowerShell?

### Explanation

Per Microsoft, PowerShell is a cross-platform task automation solution combining a command-line shell, a scripting language, and a configuration management framework. It's built on the .NET framework and, critically, it's **object-oriented** rather than purely text-based, which lets it handle complex data types and interact with system components far more effectively than older tools.

**A bit of history:** PowerShell was created because older Windows tools like `cmd.exe` and batch scripts couldn't keep up with the demands of managing complex enterprise environments. Microsoft engineer Jeffrey Snover recognised a fundamental difference between Windows and Unix: Windows relies on structured data and APIs, while Unix traditionally treats everything as plain text. This made directly porting Unix-style tools to Windows impractical. His solution was to build PowerShell around an object-oriented model instead, combining the simplicity of scripting with the depth of the .NET framework. It was released in 2006. As cross-platform environments became more common, Microsoft released PowerShell Core in 2016 — an open-source version that runs on Windows, macOS, and Linux.

**Why "objects" matter:** In programming, an object has properties (data describing it) and methods (actions it can perform) — for example, a `car` object might have a `Color` property and a `Drive()` method. PowerShell cmdlets return full objects rather than plain text. This is the key difference from the traditional Command Prompt: a text-based command's output has to be parsed as strings if you want to extract specific information, whereas a PowerShell object already carries its data and structure, making it far easier to filter, sort, or manipulate without extra parsing steps.

### Questions

**Question:** What do we call the advanced approach used to develop PowerShell?

**Answer:** An object-oriented approach.

## Task 3 — PowerShell Basics

### Explanation

I connected to the lab machine via Remmina (SSH client) using the AttackBox's GUI, entering the target IP and the provided credentials.

**Launching PowerShell:** on a full GUI system, PowerShell can be opened via the Start Menu search, the Run dialog (`Win + R` then `powershell`), typing `powershell` into File Explorer's address bar, or via Task Manager's "Run new task." Since I only had Command Prompt access to the target, I launched PowerShell from within `cmd.exe` simply by typing `powershell` and pressing Enter, which drops into a `PS C:\...>` prompt.

**Verb-Noun syntax:** PowerShell commands are called **cmdlets** ("command-lets"), and they follow a consistent `Verb-Noun` naming pattern — the verb describes the action, the noun describes what it acts on. For example, `Get-Content` retrieves (gets) a file's content, and `Set-Location` changes (sets) the current directory. This consistency makes cmdlets far more predictable to guess or remember than traditional Windows commands.

```text
Get-Command
```

**Explanation:** Lists all available cmdlets, functions, aliases, and scripts in the current session — essential for discovering what's available to use. The output can be filtered by property, for example restricting to only functions:

```text
Get-Command -CommandType "Function"
```

**Explanation:** `-CommandType` filters the results to only the specified type of command (here, `Function`), rather than showing every cmdlet, alias, and function at once.

```text
Get-Help Get-Date
```

**Explanation:** Shows detailed usage information for a specific cmdlet — its syntax, a description, and related links. Appending `-examples` (e.g. `Get-Help Get-Date -examples`) shows common real-world usage examples for that cmdlet, which is often the fastest way to learn how to actually use something new.

```text
Get-Alias
```

**Explanation:** Lists all aliases currently available — shortcuts or alternative names that map to full cmdlets, included specifically to make the transition easier for people already used to traditional Windows or Unix commands. For example, `dir` is an alias for `Get-ChildItem`, and `cd` is an alias for `Set-Location`.

I also learned about extending PowerShell's functionality with modules downloaded from online repositories like the PowerShell Gallery:

```text
Find-Module -Name "PowerShell*"
```

**Explanation:** Searches an online repository for modules matching a name pattern. The `*` wildcard lets me search for a partial name if I don't know the exact module name.

```text
Install-Module -Name "PowerShellGet"
```

**Explanation:** Downloads and installs a module from the repository, making any new cmdlets it contains available for use. Note: this requires internet access, which the lab machine doesn't have, so this specific command wasn't runnable in this environment — but it's an important capability to know about for real-world use.

### Questions

**Question:** How would you retrieve a list of commands that start with the verb Remove?

**Answer:** `Get-Command -Verb Remove`

**Question:** What cmdlet has its traditional counterpart echo as an alias?

**Answer:** `Write-Output` — `echo` is an alias for `Write-Output`.

**Question:** What is the command to retrieve some example usage for the cmdlet New-LocalUser?

**Answer:** `Get-Help New-LocalUser -Examples`

## Task 4 — Navigating the Filesystem and Working with Files

### Explanation

PowerShell provides its own set of cmdlets for filesystem navigation and file management, many of which map to familiar Command Prompt (or Unix) equivalents.

```text
Get-ChildItem
```

**Explanation:** Lists files and directories at a given path (set with `-Path`), or the current directory if no path is given. This is PowerShell's equivalent of `dir` in Command Prompt or `ls` in Unix.

```text
Set-Location -Path ".\Documents"
```

**Explanation:** Changes the current working directory — PowerShell's equivalent of `cd`.

Unlike Command Prompt, which uses separate commands for files versus directories (`mkdir`/`rmdir` for directories, no dedicated file-creation command), PowerShell unifies file and directory management under a single set of cmdlets:

```text
New-Item -Path ".\captain-cabin\captain-wardrobe" -ItemType "Directory"
```

**Explanation:** Creates a new item at the given path. `-ItemType` specifies what kind of item to create — `"Directory"` for a folder, or `"File"` for a file. This single cmdlet replaces the need for separate directory- and file-creation commands.

```text
Remove-Item -Path ".\captain-cabin\captain-wardrobe\captain-boots.txt"
```

**Explanation:** Deletes the specified item, whether it's a file or a directory — again, one cmdlet covers both cases, unlike Command Prompt's separate `del` and `rmdir`.

```text
Copy-Item -Path .\captain-cabin\captain-hat.txt -Destination .\captain-cabin\captain-hat2.txt
```

**Explanation:** Copies a file (or directory) to a new location, PowerShell's equivalent of `copy`.

```text
Move-Item
```

**Explanation:** Moves a file or directory to a new location, PowerShell's equivalent of `move`.

```text
Get-Content -Path ".\captain-hat.txt"
```

**Explanation:** Reads and displays the contents of a file — PowerShell's equivalent of `type` in Command Prompt or `cat` in Unix.

### Questions

**Question:** What cmdlet can you use instead of the traditional Windows command type?

**Answer:** `Get-Content`

**Question:** What PowerShell command would you use to display the content of the "C:\Users" directory?

**Answer:** `Get-ChildItem -Path C:\Users`

**Question:** How many items are displayed by the command described in the previous question?

**Answer:** *This depends on the actual contents of `C:\Users` on your deployed lab machine, which I don't have recorded. Let me know the count from your `Get-ChildItem` output and I'll add it here.*

## Task 5 — Piping, Filtering, and Sorting Data

### Explanation

**Piping** (`|`) sends the output of one command directly into the next as input, letting several commands be chained into a single sequence. This concept exists in both traditional Windows CLI and Unix shells, but PowerShell's version is more powerful because it pipes actual **objects** (with their full properties and methods intact) rather than plain text, meaning the receiving cmdlet can act on structured data immediately without needing to parse text first.

```text
Get-ChildItem | Sort-Object Length
```

**Explanation:** `Get-ChildItem` retrieves the files as objects, and pipes them into `Sort-Object`, which sorts them — here, by their `Length` (size) property.

```text
Get-ChildItem | Where-Object -Property "Extension" -eq ".txt"
```

**Explanation:** `Where-Object` filters incoming objects based on a condition — here, keeping only items whose `Extension` property equals (`-eq`) `.txt`.

A set of comparison operators is used with `Where-Object` and similar cmdlets, shared conceptually with other scripting languages:

- `-eq` — equal to
- `-ne` — not equal to
- `-gt` — greater than (strict; excludes values that are exactly equal)
- `-ge` — greater than or equal to
- `-lt` — less than (strict)
- `-le` — less than or equal to

Properties can also be filtered by pattern matching rather than exact value, using `-like`:

```text
Get-ChildItem | Where-Object -Property "Name" -like "ship*"
```

**Explanation:** Keeps only items whose `Name` property matches the given pattern — here, anything starting with `ship`, using `*` as a wildcard.

```text
Get-ChildItem | Select-Object Name,Length
```

**Explanation:** `Select-Object` narrows down the output to only the specified properties (here, `Name` and `Length`), rather than displaying every property of each object — useful for trimming output down to exactly what's relevant.

A pipeline isn't limited to just two cmdlets — it can be extended further, chaining sorting, filtering, and selecting together to build a precise result (for example, sorting files by size and then selecting just the largest one).

```text
Select-String -Path ".\captain-hat.txt" -Pattern "hat"
```

**Explanation:** Searches for a text pattern within a file, similar to `grep` in Unix or `findstr` in Command Prompt — useful for finding specific content inside log files or documents. `Select-String` also fully supports regular expressions for more advanced pattern matching.

### Questions

**Question:** How would you retrieve the items in the current directory with size greater than 100?

**Answer:** `Get-ChildItem | Where-Object -Property Length -gt 100`

## Task 6 — System and Network Information

### Explanation

PowerShell provides cmdlets that retrieve far more detailed system and network information than their traditional Command Prompt counterparts.

```text
Get-ComputerInfo
```

**Explanation:** Retrieves comprehensive system information in one command — OS details, hardware specs, BIOS information, and more. Its traditional counterpart, `systeminfo`, only returns a smaller subset of this same information, making `Get-ComputerInfo` considerably more thorough.

```text
Get-LocalUser
```

**Explanation:** Lists all local user accounts on the system, showing username, whether the account is enabled, and its description. This is genuinely useful for security auditing purposes — spotting accounts that shouldn't be enabled, or unexpected accounts entirely, is a basic but important part of assessing a machine's local security posture.

```text
Get-NetIPConfiguration
```

**Explanation:** Shows network interface configuration — IP addresses, DNS servers, and gateway settings — similar in purpose to `ipconfig`, but returned as structured objects rather than plain text.

```text
Get-NetIPAddress
```

**Explanation:** Shows detail on every IP address configured on the system across all interfaces, including ones that aren't currently active — useful when `Get-NetIPConfiguration`'s summary isn't detailed enough.

For this task, I used `Get-LocalUser` to check which accounts existed on the lab machine beyond the default Administrator and my own logged-in user, and found an additional enabled account with a suspicious description set on it. I then navigated to that user's home folder under `C:\Users` to look for further clues, using `Get-ChildItem` and `Get-Content` to explore and read files inside it.

### Questions

**Question:** Other than your current user and the default "Administrator" account, what other user is enabled on the lab machine?

**Answer:** *This is specific to your `Get-LocalUser` output on the deployed VM, which I don't have recorded. Let me know the username shown as Enabled: True and I'll add it here.*

**Question:** This lad has hidden his account among the others with no regard for our beloved captain! What is the motto he has so bluntly put as his account's description?

**Answer:** *This is the Description value from your `Get-LocalUser` output for that account — I don't have it recorded. Let me know what it says and I'll add it here.*

**Question:** Can you navigate the filesystem and find the hidden treasure inside this pirate's home?

**Answer:** *This depends on what you actually found inside that user's home folder on your deployed VM — I don't have that content. Let me know what you found and I'll add it here.*

## Task 7 — Real-Time System Analysis

### Explanation

This task moved beyond static machine details into dynamic, real-time system information — running processes, services, active network connections, and file integrity — the kind of data that matters most during troubleshooting, incident response, and threat hunting.

```text
Get-Process
```

**Explanation:** Lists every currently running process, along with details like handle count, memory usage, and CPU time. Useful for spotting resource-heavy or unfamiliar processes.

```text
Get-Service
```

**Explanation:** Lists all services on the machine along with their status (Running, Stopped, Paused) and display name. This is used heavily by system administrators for troubleshooting, but just as importantly by forensic analysts hunting for anomalous or maliciously installed services — a technique often used for persistence by attackers.

```text
Get-NetTCPConnection
```

**Explanation:** Displays current TCP connections, including local and remote addresses/ports, connection state, and the owning process. This is particularly valuable during incident response or malware analysis, since it can reveal hidden backdoors or active connections out to an attacker-controlled server that wouldn't be obvious just from looking at running processes alone.

```text
Get-FileHash -Path .\ship-flag.txt
```

**Explanation:** Generates a cryptographic hash (SHA256 by default) of a file. This is a core technique in incident response, threat hunting, and malware analysis for verifying file integrity — comparing a file's hash against a known-good value (or a known-malicious one) can confirm whether a file has been tampered with, without needing to inspect its full contents.

I also learned about checking a file's **Alternate Data Streams (ADS)** — a Windows NTFS feature I'd first come across back in Windows Fundamentals 1, where extra hidden data can be attached to a file beyond its normal visible contents:

```text
Get-Item -Path "C:\House\house_log.txt" -Stream *
```

**Explanation:** Lists every data stream attached to a file. Every NTFS file has a default `:$DATA` stream (its normal, visible contents), but this command will also reveal any additional named streams — which is exactly the kind of place malware has historically been known to hide data, since ADS content isn't visible through Windows Explorer or normal directory listings.

Following on from the hidden user account I found in the previous task, I used `Get-FileHash` to hash the file containing the treasure I found in that user's home folder, and used `Get-Service` to track down a service on the machine whose `DisplayName` had been tampered with to display the same suspicious motto found in that user's account description.

### Questions

**Question:** In the previous task, you found a marvellous treasure carefully hidden in the lab machine. What is the hash of the file that contains it?

**Answer:** *This depends on the actual file you found and its `Get-FileHash` output on your deployed VM — I don't have that recorded. Let me know the hash and I'll add it here.*

**Question:** What property retrieved by default by Get-NetTCPConnection contains information about the process that has started the connection?

**Answer:** `OwningProcess`

**Question:** Some vital service has been installed on this pirate ship... can you find the service name?

**Answer:** *This depends on which service's `DisplayName` was tampered with on your deployed VM, found via `Get-Service` — I don't have that recorded. Let me know the service name and I'll add it here.*

## Task 8 — Scripting

### Explanation

**Scripting** is writing a sequence of commands into a text file (a script) so they can be executed automatically, rather than typed manually one at a time. It's essentially giving the computer a to-do list to work through on its own — saving time, cutting down on manual errors, and making it possible to automate tasks that would be too tedious or complex to do by hand every time.

While learning to actually write PowerShell scripts was outside the scope of this room, understanding *why* scripting matters in cybersecurity is important across every role:

- **Blue team** (incident responders, malware analysts, threat hunters) — scripts can automate log analysis, anomaly detection, extracting indicators of compromise (IOCs), reverse-engineering malware, and scanning systems for signs of intrusion.
- **Red team** (penetration testers, ethical hackers) — scripts can automate system enumeration, execute remote commands, and even craft obfuscated scripts specifically to bypass defensive tooling, simulating real attacker behaviour to test a system's resilience.
- **System administrators** — scripts can automate compliance checks, manage configurations at scale, enforce security policies, monitor system health, and even respond automatically to security incidents.

One especially important cmdlet for remote work and automation is `Invoke-Command`:

```text
Invoke-Command -FilePath c:\scripts\test.ps1 -ComputerName Server01
```

**Explanation:** Runs a script that exists locally on my machine against a specified remote computer, with the results returned back to my local machine. `-FilePath` points to the local script; `-ComputerName` specifies the target.

```text
Invoke-Command -ComputerName Server01 -Credential Domain01\User01 -ScriptBlock { Get-Culture }
```

**Explanation:** Runs a single command (or block of commands) directly on a remote computer without needing a saved script file at all. `-Credential` specifies which account to authenticate as on the remote machine (PowerShell will then prompt for that account's password), and `-ScriptBlock { ... }` contains the actual command(s) to run remotely — in this example, `Get-Culture`. The result is exactly as if I'd typed that command directly into a PowerShell session on the remote machine itself.

`Invoke-Command` is fundamental for legitimate remote administration and automation across many machines at once, but it's worth being clear-eyed that the exact same capability is what makes it such a common tool for penetration testers — and real attackers — to execute commands or payloads on a target system during an engagement.

### Questions

**Question:** What is the syntax to execute the command Get-Service on a remote computer named "RoyalFortune"? Assume you don't need to provide credentials to establish the connection.

**Answer:** `Invoke-Command -ComputerName RoyalFortune -ScriptBlock { Get-Service }`

## Task 9 — Conclusion

### Explanation

The room wrapped up by confirming that, having worked through PowerShell's core cmdlets, filesystem management, piping/filtering, and system/network/process investigation, I now have a solid enough toolkit to explore even fairly locked-down corners of a Windows system. The natural next step in the Command Line module is the Linux Shells room, to build the equivalent skillset on the Linux side.

## Key Takeaways

- Learned that PowerShell is object-oriented rather than text-based, which is the core reason it's more powerful than traditional Command Prompt for data manipulation.
- Understood the consistent Verb-Noun naming convention used by all cmdlets, and how `Get-Command`, `Get-Help`, and `Get-Alias` work together as discovery tools for learning new cmdlets.
- Learned PowerShell's unified approach to file/directory management (`New-Item`, `Remove-Item`, `Copy-Item`, `Move-Item`, `Get-Content`) compared to Command Prompt's separate commands for each.
- Practiced building pipelines with `Sort-Object`, `Where-Object`, `Select-Object`, and `Select-String` to filter and refine command output using real comparison and pattern-matching operators.
- Learned how `Get-ComputerInfo`, `Get-LocalUser`, `Get-NetIPConfiguration`, and `Get-NetIPAddress` provide far richer system and network detail than their Command Prompt equivalents.
- Practiced a practical security-relevant workflow: using `Get-LocalUser` to spot an unexpected enabled account, then navigating its home directory to investigate further.
- Learned how `Get-Process`, `Get-Service`, `Get-NetTCPConnection`, and `Get-FileHash` support real-time monitoring, incident response, and file integrity verification, including a hands-on tampered-service investigation.
- Understood why scripting and `Invoke-Command` are essential across blue team, red team, and sysadmin roles alike, since the same remote-execution power that enables automation is also what makes PowerShell a frequent target for attacker abuse.

## Conclusion

This room gave me a solid working foundation in PowerShell, building directly on the Command Prompt skills from the previous room. The object-oriented pipeline model (`Get-ChildItem | Where-Object | Sort-Object`, for example) is the single biggest shift from traditional CLI thinking, and the real-time investigation tools in Task 7 (`Get-Process`, `Get-Service`, `Get-NetTCPConnection`, `Get-FileHash`) made it clear just how directly PowerShell applies to actual incident response and threat hunting work, not just day-to-day administration. Combined with the scripting and `Invoke-Command` overview in Task 8, I can see clearly why PowerShell is both such a powerful legitimate admin tool and such a common target for abuse in real attacks — the same cmdlets I used here to audit local users, hunt down a tampered service, and explore the filesystem are exactly the kind of reconnaissance and remote execution an attacker would rely on after gaining initial access to a Windows host.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
