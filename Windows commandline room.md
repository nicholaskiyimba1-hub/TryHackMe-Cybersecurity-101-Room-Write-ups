# Windows Command Line

**Write-up by: Nicholas Kiyimba**

## Introduction

This room focuses on `cmd.exe`, the default command-line interpreter in Windows, and teaches how to use it to display system information, check and troubleshoot network configuration, manage files and folders, and check running processes.

A CLI has real advantages over a GUI once you're past the learning curve: it's faster for repetitive tasks (no need to click through menus every time), uses fewer system resources, is much easier to automate through scripts and batch files, and is well suited for remotely managing systems like servers, routers, or IoT devices, especially over slower connections. This matters in cybersecurity specifically because a huge amount of real-world work, whether administering systems or investigating a compromised one, happens over a command-line connection rather than a full desktop session, so being comfortable with `cmd.exe` is a baseline skill.

## Task 1 — Introduction

### Explanation

I started the lab machine and the AttackBox, then connected from the AttackBox's terminal to the target VM over SSH:

```bash
ssh user@MACHINE_IP
```

Since this was the first connection to this target, I was prompted to trust the host (answered yes), then entered the password when prompted (no characters appear on screen while typing it, which is normal SSH behaviour).

### Questions

**Question:** What is the default command line interpreter in the Windows environment?

**Answer:** `cmd.exe` (Command Prompt).

## Task 2 — Basic System Information

### Explanation

Commands can only be run from within the Windows **Path** — the set of directories Windows searches through to find the executable for a command I type. I can check the current path with:

```text
set
```

**Explanation:** Displays all current environment variables, including the line starting with `Path=`, which lists every directory Windows searches when I run a command by name.

To check the OS version specifically:

```text
ver
```

**Explanation:** Prints the exact Windows version and build number, e.g. `Microsoft Windows [Version 10.0.17763.1821]`.

For a much more detailed system overview:

```text
systeminfo
```

**Explanation:** Lists detailed information about the system, including OS name and version, manufacturer, configuration, and hardware/processor details. This is a single command that pulls together information I would otherwise have to check in several different GUI locations (System Information, About, Device Manager, etc.).

I also picked up two general tips here:

- Any command's output can be piped through `more` if it's too long to fit on one screen, e.g. `driverquery | more`, which pages the output and lets me scroll through it with the space bar, or exit early with `Ctrl + C`.
- `help` shows help information for a specific command, and `cls` clears the screen.

### Questions

**Question:** What is the OS version of the Windows VM?

**Answer:** *This is specific to your deployed VM's `ver` output — I don't have that recorded. Let me know the version string shown and I'll add it here.*

**Question:** What is the hostname of the Windows VM?

**Answer:** *This is specific to your deployed VM's `systeminfo`/`Host Name` output — I don't have that recorded. Let me know the hostname and I'll add it here.*

## Task 3 — Network Troubleshooting

### Explanation

This task covered the core command-line tools for checking and troubleshooting network configuration.

```text
ipconfig
```

**Explanation:** Shows the current network configuration for each adapter, including the IPv4 address, subnet mask, and default gateway.

```text
ipconfig /all
```

**Explanation:** Extends the basic output with more detail — DNS servers, whether DHCP is enabled, the DHCP lease start/expiry times, and the adapter's physical (MAC) address.

```text
ping target_name
```

**Explanation:** Sends ICMP packets to the target and listens for replies, to confirm the target is reachable. The output also reports round-trip time statistics and packet loss, which is useful for judging connection quality, not just reachability.

```text
tracert target_name
```

**Explanation:** Traces the actual network path (each router/hop) a packet takes to reach the target, by relying on routers along the way reporting back once a packet's time-to-live (TTL) expires. Useful for identifying exactly where a connection is failing or slowing down, rather than just knowing the destination is unreachable.

```text
nslookup example.com
```

**Explanation:** Looks up the IP address(es) associated with a domain name, using the system's default DNS server. Providing a second argument, e.g. `nslookup example.com 1.1.1.1`, forces the lookup to use a specific DNS server instead (in this case, Cloudflare's `1.1.1.1`) — useful for comparing whether different DNS servers are returning different results, which can indicate DNS poisoning or misconfiguration.

```text
netstat
```

**Explanation:** Shows currently active network connections. Run with no arguments, it lists established connections along with local/foreign addresses and connection state.

I then looked at some useful `netstat` options (found via `netstat -h` for the full help page):

- `-a` — shows all connections and listening ports, not just established ones
- `-b` — shows the executable/program responsible for each listening port or connection
- `-o` — shows the process ID (PID) associated with each connection
- `-n` — displays addresses and ports numerically rather than resolving names

Combined as `netstat -abon`, this gives a very complete picture: for example, being able to see that `sshd.exe` is the process listening on port 22, along with its PID. From a security standpoint, this combination is genuinely useful for spotting unexpected listening ports or unfamiliar processes accepting connections on a machine.

### Questions

**Question:** Which command can we use to look up the server's physical address (MAC address)?

**Answer:** `ipconfig /all` — it lists the Physical Address (MAC address) for each network adapter.

**Question:** What is the name of the service listening on port 135?

**Answer:** `svchost.exe` (associated with the RPC service, RpcSs).

**Question:** What is the name of the service listening on port 3389?

**Answer:** *Port 3389 is the standard RDP port, so this is most likely a Remote Desktop related service — but the exact process/service name depends on your VM's actual `netstat -abon` output, which I don't have recorded. Let me know what it shows for port 3389 and I'll confirm this.*

## Task 4 — File and Disk Management

### Explanation

This task covered navigating directories and managing files entirely from the command line.

**Directories:**

```text
cd
```

**Explanation:** With no parameters, shows the current drive and directory — essentially "where am I?".

```text
dir
```

**Explanation:** Lists the contents (files and subdirectories) of the current directory, along with file sizes, dates, and free disk space. Two useful options:

- `dir /a` — includes hidden and system files, which are excluded by default
- `dir /s` — lists files in the current directory and every subdirectory beneath it

```text
tree
```

**Explanation:** Displays a visual, indented tree of the current directory's subdirectory structure, which is a quicker way to understand a folder's layout than repeated `dir` commands.

```text
cd target_directory
```

**Explanation:** Changes into the specified directory, the command-line equivalent of double-clicking a folder. `cd ..` moves up one level to the parent directory.

```text
mkdir directory_name
```

**Explanation:** Creates a new directory ("make directory").

```text
rmdir directory_name
```

**Explanation:** Removes/deletes a directory ("remove directory").

**Files:**

```text
type filename
```

**Explanation:** Prints the contents of a text file directly to the screen. Best suited to short files that fit within the terminal window.

```text
more filename
```

**Explanation:** Displays a text file one page at a time rather than dumping everything at once, better suited to long files — Space moves forward a page, Enter moves forward a line.

```text
copy source destination
```

**Explanation:** Copies a file to a new location or name, leaving the original in place. For example, `copy test.txt test2.txt` creates a duplicate named `test2.txt` in the same directory.

```text
move source destination
```

**Explanation:** Moves a file to a new location (or renames it), removing it from the original location. For example, `move test2.txt ..` moves the file up one directory level.

```text
del filename
```
or
```text
erase filename
```

**Explanation:** Deletes a file. `del` and `erase` are interchangeable.

I also learned that the wildcard character `*` can be used to match multiple files at once, for example `copy *.md C:\Markdown` copies every file with a `.md` extension into the `C:\Markdown` directory in one go, rather than copying each file individually.

### Questions

**Question:** What are the file's contents in `C:\Treasure\Hunt`?

**Answer:** *This depends on the actual file(s) present in that directory on your deployed VM, which I don't have — this would need to be read directly with `type` or `more` on your machine. Let me know the contents and I'll add them here.*

## Task 5 — Task and Process Management

### Explanation

This task covered achieving the same kind of process-management functionality Task Manager provides, but from the command line.

```text
tasklist
```

**Explanation:** Lists every running process, along with its Image Name, PID, Session Name, Session number, and memory usage — essentially the Processes tab of Task Manager, in text form.

Since this output can be very long, filtering is useful. The full set of available filters can be checked with `tasklist /?`. To filter for a specific process by name:

```text
tasklist /FI "imagename eq sshd.exe"
```

**Explanation:** `/FI` applies a filter to the output — in this case, restricting results to only processes where the image name equals `sshd.exe`. This is much faster than scrolling through the full list looking for a specific process, especially on a busy system.

Once I have a specific PID (from `tasklist`), I can terminate that process directly:

```text
taskkill /PID target_pid
```

**Explanation:** Sends a termination request to the process with the given PID. This is the command-line equivalent of selecting a process in Task Manager and clicking "End Task."

### Questions

**Question:** What command would you use to find the running processes related to `notepad.exe`?

**Answer:** `tasklist /FI "imagename eq notepad.exe"`

**Question:** What command can you use to kill the process with PID 1516?

**Answer:** `taskkill /PID 1516`

## Task 6 — Conclusion

### Explanation

The room wrapped up by mentioning a few additional commands that exist but weren't covered in depth, since they were considered outside the scope of a beginner-focused room:

- `chkdsk` — checks the file system and disk volumes for errors and bad sectors
- `driverquery` — lists installed device drivers
- `sfc /scannow` — scans system files for corruption and attempts to repair them

The room reinforced two important habits: appending `/?` to most commands shows a help page for that command, and `more` is useful both for reading text files directly (`more file.txt`) and for paging through long command output (`some_command | more`).

### Questions

**Question:** The command `shutdown /s` can shut down a system. What is the command you can use to restart a system?

**Answer:** `shutdown /r`

**Question:** What command can you use to abort a scheduled system shutdown?

**Answer:** `shutdown /a`

## Key Takeaways

- Learned why the command line is often faster and more resource-efficient than a GUI, and why it's the standard way to manage remote systems.
- Learned how to pull basic and detailed system information (`ver`, `systeminfo`) and check the command search path (`set`).
- Practiced core network troubleshooting commands: `ipconfig`/`ipconfig /all`, `ping`, `tracert`, `nslookup`, and `netstat` (including useful flag combinations like `-abon` for tying a listening port back to its owning process).
- Learned the full set of file and directory management commands (`cd`, `dir`, `tree`, `mkdir`, `rmdir`, `type`, `more`, `copy`, `move`, `del`/`erase`) and how to use wildcards to act on multiple files at once.
- Learned how to list, filter, and terminate running processes directly from the command line using `tasklist` and `taskkill`.
- Understood that `/?` and `more` are two habits worth carrying into any new or unfamiliar command.

## Conclusion

This room gave me practical, hands-on command-line skills for administering a Windows machine without ever touching the GUI, from checking system and network configuration through to managing files and killing misbehaving processes. Being able to gather this same information entirely from a terminal (especially over SSH, as I did to connect to the lab machine in the first place) is directly useful for remote administration and troubleshooting, and I can see how commands like `netstat -abon` and `tasklist` would be just as relevant in an investigative or incident-response context as in day-to-day admin work. This sets me up well for moving on to the Windows PowerShell room next.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
