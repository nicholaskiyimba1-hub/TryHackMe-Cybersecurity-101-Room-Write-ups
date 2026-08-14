# TryHackMe Write-up: Windows Fundamentals Part 2

**Write-up by: Nicholas Kiyimba**

## Introduction

This is my write-up for Windows Fundamentals Part 2 on TryHackMe. Following on from Part 1, where I covered the desktop, UAC, the Control Panel, Settings, and Task Manager, this room went into more advanced Windows utilities, mostly ones accessible through the System Configuration panel (MSConfig).

## Task 1: Introduction and Lab Setup

I started the attached lab machine and connected via Remote Desktop using the administrator credentials provided, accepting the certificate warning to get into the remote system, the same process as in Part 1.

## Task 2 & 3: System Configuration and UAC Settings

System Configuration (`msconfig`) is a utility built for advanced troubleshooting, mainly used to diagnose startup issues. Opening it requires local administrator rights. It has five tabs:

- **General** – controls what services and devices load at boot, with options for Normal, Diagnostic, or Selective startup
- **Boot** – defines boot options for the operating system
- **Services** – lists every configured service on the system, whether running or stopped (a service is a special type of application that runs in the background)
- **Startup** – on a normal Windows 10/11 machine this would list startup programs, but Microsoft recommends using Task Manager for this instead, since MSConfig isn't meant to be a startup manager. On the Windows Server lab machine specifically, startup items don't show up in Task Manager or this tab at all. Instead, the only reliable way to check user-level startup items on a server is to open the Startup folder directly, done by pressing `Win + R`, typing `shell:startup`, and pressing Enter.
- **Tools** – a list of various diagnostic and configuration tools, each with a short description and a "Selected command" box showing the exact command used to launch it

I also looked at the User Account Control settings, which I'd already covered conceptually in Part 1. The UAC slider has four levels:

- **Always notify** – the highest security level; Windows dims the desktop and notifies for any change, whether made by an app or by me
- **Notify for apps** – the default setting; only notifies when apps try to make changes, not when I change Windows settings myself
- **Notify without dimming** – the same as "Notify for apps" but without dimming the screen
- **Never notify** – turns off all UAC notifications, which is not recommended

The command to open the UAC settings window directly is `UserAccountControlSettings.exe`.

I also explored Advanced System Settings, found by searching "View advanced system settings". Under the Performance section, I could view and adjust the page file, which Windows uses as extra virtual memory when physical RAM runs low. This showed me the drive the page file is stored on, its initial and maximum size, and whether Windows manages the size automatically. Under Startup and Recovery, I found the crash dump settings, which control how much diagnostic information Windows saves when the system crashes (a Blue Screen of Death). The options range from no dump at all up to a complete memory dump, and this information can help administrators figure out what caused a crash.

## Task 4: Computer Management

Computer Management (`compmgmt.msc`) brings together several tools under three main sections: System Tools, Storage, and Services and Applications.

**System Tools** includes:

- **Task Scheduler** – lets me view, create, and manage tasks that run automatically, either on a schedule, at login/logoff, or at a specific one-time trigger. The Task Scheduler Library shows all scheduled tasks and their triggers.
- **Event Viewer** – shows a record of events that have happened on the computer, split into a tree of event log providers on the left, an overview pane in the middle, and an actions pane on the right. This is a key tool for diagnosing problems or investigating what happened on a system.
- **Shared Folders** – shows all shares on the machine, including default Windows shares like `C$` and administrative shares like `ADMIN$`, along with which sessions are currently connected and which files they have open.
- **Local Users and Groups** – the same `lusrmgr.msc` tool I used back in Windows Fundamentals 1.
- **Performance** – contains Performance Monitor (`perfmon`), used to view real-time or logged performance data for troubleshooting.
- **Device Manager** – lets me view and configure the hardware attached to the computer, including disabling a device.

**Storage** includes Windows Server Backup and Disk Management. Disk Management is used for tasks like setting up a new drive, extending or shrinking a partition, or changing a drive letter. Since the lab machine runs Windows Server, some of these storage tools aren't present on a typical Windows 10/11 machine.

**Services and Applications** shows all services on the system along with their status. Right-clicking a service and viewing its properties reveals more detail, such as its actual service name (which can differ from its display name), the path to its executable, and its startup type. A service's startup type can be:

- **Automatic** – starts every time the system boots
- **Manual** – only starts when triggered by another process or user
- **Disabled** – does not run at all

This section also gives access to WMI (Windows Management Instrumentation) control, which allows scripting languages like PowerShell to manage Windows machines locally or remotely.

## Task 5: System Information

System Information (`msinfo32`) gathers detailed information about the computer's hardware, system components, and installed software, useful for diagnosing issues. It's split into three sections:

- **Hardware Resources** – lower-level technical details, mostly relevant to advanced troubleshooting
- **Components** – information about installed hardware devices, such as Display and Input devices
- **Software Environment** – information about installed software, Environment Variables, and Network Connections

I revisited Environment Variables here, which I'd briefly touched on in Windows Fundamentals 1 in relation to the `%windir%` variable. I found that `ComSpec`, one of the environment variables, points to the path of the command interpreter (`cmd.exe`). Environment Variables can also be reached without opening `msinfo32`, through Control Panel > System and Security > System > Advanced system settings, or through Settings > System > About > Advanced system settings.

I also tried the search bar near the bottom of the System Information window, searching within Components for "IP address" to quickly find the relevant network details without manually browsing the whole tree.

## Task 6: Resource Monitor

Resource Monitor (`resmon`) shows detailed, per-process and overall CPU, memory, disk, and network usage. It's aimed at advanced troubleshooting, and can also be used to start, stop, pause, or resume services, and to close unresponsive applications.

The Overview tab is split into four sections, each with its own dedicated tab for more detail:

- CPU
- Memory
- Disk
- Network

There's also a live graphical panel on the right-hand side showing real-time usage for each of these areas.

## Task 7: Command Prompt

This task introduced the Command Prompt (`cmd`), which was the primary way of interacting with computers before graphical interfaces existed. Even with a GUI available, the command line is still very useful for quickly getting information about a system.

I covered a few basic commands:

- `hostname` – shows the computer's name
- `whoami` – shows the currently logged-in user
- `ipconfig` – shows the network address settings for the computer. Adding the `/all` parameter shows more detailed information, including things like the MAC address and DNS servers.
- `cls` – clears the command prompt screen

Most commands support a help manual accessed by appending `/?`, for example `ipconfig /?`. I also looked at `netstat`, which displays protocol statistics and current TCP/IP network connections, and changes its output depending on which parameters are added, such as `-a`, `-b`, or `-e`.

The `net` command was a bit different since it works with sub-commands (used to manage network resources). Running `net` alone shows the available sub-commands, but to get help on a specific one, the syntax is `net help sub-command` rather than the usual `/?`. For example, `net help user` shows help for managing user accounts through the command line.

The full command to open the Internet Protocol Configuration through System Configuration's Tools tab is `ipconfig`.

## Task 8: Registry Editor

The Windows Registry is a central database that stores configuration information Windows needs to run: user profiles, installed applications and their file associations, hardware information, ports in use, and more. Windows constantly reads from the registry during normal operation.

The Registry Editor (`regedit`) is one of the main ways to view and edit the registry directly. I was reminded that this is meant for advanced users only, since incorrect changes to the registry can cause real problems with how the computer runs, so I avoided making any changes myself and just looked at how the tool is structured.

## What I Learned

By the end of this room, I had a much better understanding of:

- The System Configuration (MSConfig) utility and its five tabs, including how startup management differs on Windows Server
- How to check and change UAC notification levels
- The tools available through Computer Management: Task Scheduler, Event Viewer, Shared Folders, Performance Monitor, Device Manager, Disk Management, and Services
- How to use System Information (`msinfo32`) to view hardware, component, and software details, including environment variables
- How to monitor live system resource usage with Resource Monitor
- Basic but useful Command Prompt commands for gathering system and network information
- What the Windows Registry is and why it needs to be handled carefully

This room gave me a much deeper toolkit for exploring and troubleshooting a Windows system beyond what I covered in Part 1, and a lot of these utilities (Task Scheduler, Event Viewer, Registry Editor, Command Prompt) are ones I expect to come back to often in future, more security-focused rooms. I completed this room myself as part of my personal TryHackMe learning path.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
