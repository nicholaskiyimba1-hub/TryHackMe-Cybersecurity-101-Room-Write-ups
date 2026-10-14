# Log Fundamentals: Windows Event Logs and Web Server Access Logs

## Introduction

This writeup documents what I learned in the TryHackMe Log Fundamentals room. The room introduces logs as a source of evidence on a computer system, explains the main categories of logs, and then gives two practical exercises. In the first I used Windows Event Viewer to investigate account activity on a Windows machine, and in the second I read an Apache web server access log manually from the Linux AttackBox using command line tools. I wrote this to keep a clear record of the concepts and the commands so that I can return to them later and still understand why each step was done.

## What a log is and why it matters

The room begins with an analogy to a crime scene, where footprints in the snow let an investigator reconstruct what someone did. A log works in a similar way. It is a record of an event that happened on a system, so a collection of logs becomes a digital paper trail of activity. Starting the computer produces a log entry, logging in produces one, deleting a user account produces one, and creating a user and giving that user administrator privileges produces several. Because so much activity leaves a trace, logs are the place where most traces of an attack are found in a digital system.

Logs were not originally created for security. Early programs wrote errors and status messages so that developers could understand why something was failing, and troubleshooting is still one of their main uses. Today logs are used for event monitoring, incident investigation and forensics, performance checks, and troubleshooting. They are also important for auditing and compliance, because many regulations require organizations to keep logs for a minimum period. For example, PCI DSS requires audit log history to be retained for at least twelve months, with the most recent three months immediately available for analysis. In my notes I was unsure about the exact retention period, so I checked this and wrote the correct requirement here.

## Types of logs

Logs can be grouped into broad categories, and these categories overlap. System logs describe the state of the system itself, such as when it started or shut down. Security logs record authentication and authorization events, for example whether a login attempt succeeded or failed. Application logs come from installed software and can record crashes, errors, or application data. Audit logs support auditing and accountability. Network logs describe incoming and outgoing traffic, and I learned that firewall logs belong in this group. Access logs record what was accessed and when, which applies to web servers, databases, and many other services. The room also made clear that these categories are a way of thinking about logs and are not strict rules, because different operating systems split them differently and an entry can reasonably fit more than one category.

Most devices have logging enabled by default, including computers, mobile devices, and IoT devices. On a personal computer the owner can usually clear logs, so an attacker who gains access can also try to delete them to hide what happened. This is one reason organizations forward logs from each machine to a centralized system, commonly a Security Information and Event Management system (SIEM). An agent on the computer sends each log to the central system as it is created, so deleting the local copy does not remove the evidence. The room also mentioned that web application firewall logs would be used, but my notes only cover the Windows and Apache exercises, so I am not claiming any firewall log analysis here.

## Windows event logs

Windows organizes its logs into its own categories, and the three I focused on were Application, System, and Security. The Security log is the most important one for security work because it records security related activity such as logins, account changes, and privilege use. The room also showed other entries in the same tool, such as Setup and Forwarded Events, but I did not use them and I am not describing them in detail.

### Event Viewer

Windows provides a graphical tool called Event Viewer for reading these logs. I first typed the wrong name when searching for it, and the correct name is Event Viewer. The window has three main areas. The left pane is where I choose which log to look at, and I used the Security log under Windows Logs. The middle pane lists the individual events and shows the full details of whichever event I select. The right pane contains actions, such as filtering the current log, finding text, saving events to a separate file for evidence, and clearing a log. Clearing is the action that an attacker might use to remove traces.

Each event contains several useful fields. The description explains what happened, the log name tells me which log it came from, the logged field gives the time the event was recorded, and the event ID identifies the kind of activity. An event ID is not unique to one single event. It identifies a type of event, so every successful logon has the same ID, a failed logon has a different one, and creating an account has another. The room listed the most important IDs, and I understood that I do not need to memorize them. They can be looked up, and SIEM dashboards usually translate them into plain language.

The event IDs that mattered in this exercise were 4624 for a successful logon, 4720 for a user account being created, 4722 for a user account being enabled, and 4724 for an attempt to reset an account password. Filtering the current log by an event ID shows only that type of event. For example, filtering for 4624 shows only successful logins.

### Investigating account activity

The scenario was that data had been exfiltrated from a file server, and I was asked to find the activity of the attacker on the compromised system before the file server was accessed. This was a guided TryHackMe exercise and not a real investigation. I used the Windows machine for this part, and the virtual machine was slow to load and needed several restarts before it worked.

I opened the Security log and filtered for event ID 4720 to find account creation events. The filter returned three events, and since I wanted the most recent account, I compared their timestamps. One detail I had to be careful about is that the event contains two account names. The Subject section shows the account that performed the action, and the New Account section shows the account that was created. Reading the event only from the top can lead to the wrong answer, so I opened the Details tab and read the target account name, which was `hacked`. The Subject account that created it was `Administrator`.

I then filtered for event ID 4722 to see when the account was enabled. Creating an account and enabling it are separate events, so I paid attention to the timestamps, and the enabled event for `hacked` appeared almost at the same moment as the creation. Finally I filtered for event ID 4724 to check for password reset attempts. There were multiple events, and one of them involved the system account changing the password of the Administrator account, which was not relevant. The relevant events were made by Administrator and targeted `hacked`, and there were two attempts about two minutes apart. From this I understood how a sequence of event IDs, read in order of time, can describe what happened to one account: it was created, enabled, and had its password reset.

## Web server access logs

A web server records information about every request it receives, because this helps with troubleshooting and also with detecting attacks such as someone sending a malicious request. On the Linux machine used in this room, the Apache web server writes these records to `/var/log/apache2/access.log`. For this practice I worked on the AttackBox, where the room files were stored in a Rooms directory in the root user's home directory. On Linux there is no graphical interface for this, so the log is read as a text file.

### Reading a log line

Each line in an Apache access log describes one request. The main parts are the client IP address, the timestamp, the request line, the status code, and the user agent. The request line contains the HTTP method, such as GET or POST, the requested URL path, and the HTTP version. The status code shows how the server responded, for example 200 for a successful request and 500 for a server error.

In my first reading of the log I misunderstood several fields, so I am correcting them here. In the common Apache combined log format, the IP address is followed by two fields that are often just a dash. The first of these is the identity of the client reported by the identd service, and the second is the username if HTTP authentication was used. Both are usually empty on modern systems. The timestamp and the request line come next, then the status code and the size of the response in bytes. After that comes the referer, which is the page the user came from, and it is a dash when there is none. The last quoted field is the user agent string. My notes treated the second field as the referer and did not recognize the trailing field correctly, so I should not rely on those notes.

The user agent string looked confusing because it contained Mozilla, AppleWebKit, Chrome, and Safari together. This happens for historical compatibility reasons, because browsers added each other's names to their user agent strings so that websites would serve them the same content. It does not mean the client was Firefox or Safari. In the line I looked at, the string contained a Windows platform token and Chrome, so the client was Chrome on Windows. I understood this at a basic level, and I have not studied the history of user agent strings in depth.

### Commands used for manual analysis

The `cat` command prints the contents of a file to the terminal. It can also take several files, which is useful when logs are rotated. Organizations often split logs into separate files, for example one for each month, so that a single file does not become too large to search or share. When an attack period crosses two files, they can be combined for analysis. The following is an example of the idea and not a result from the room, since the file names depend on how rotation is configured.

```bash
cat access.log access.log.1
```

The `grep` command searches for a pattern and prints only the matching lines. Searching for an IP address in the access log shows every request made from that address, which is a quick way to follow one client's activity.

```bash
grep "172.16.0.1" access.log
```

The address `172.16.0.1` is the IP address that the room used in its questions, and I include it only as an example of the syntax.

Printing a large log with `cat` is not practical because the output scrolls past the screen. The `less` command displays a file one page at a time, and I used it as the main tool for this exercise.

```bash
less access.log
```

Inside `less`, the space bar moves forward one page and `b` moves back one page. Pressing `/` followed by a pattern searches the file and highlights the matches. Then `n` jumps to the next match and `N` (capital) jumps to the previous match. This works like pressing Ctrl+F in a browser or spreadsheet. My original notes said that `N` was used for both directions, which was incorrect, since lowercase `n` goes forward and uppercase `N` goes backward.

### Using the log to answer questions

I searched for the `/contact` URL with `/contact` inside `less` and compared the method on each matching line, because I needed a GET request and some of the matching lines were POST requests. I then searched for the IP address from the earlier result and looked for the last POST request made by that address to read its timestamp and the URL it targeted. One thing I want to remember is the order of the entries. A normal Apache access log appends new entries at the end, so the newest request is usually at the bottom. In the practice file I read, I treated the entries near the top as the most recent, so I relied on the timestamps to confirm the order and not on position alone. Checking the timestamps is the safer habit.

## What I learned

I learned that a log is evidence of activity and that different kinds of logs answer different questions. Network logs describe traffic, security logs describe authentication and authorization, and access logs describe who requested what. I learned why logs are forwarded to a central system, since a local log can be deleted by an attacker. In Windows, I learned how to use Event Viewer to filter the Security log by event ID and why the event ID, the timestamp, and the Subject and target account fields all matter when reconstructing what happened to an account. On Linux, I learned how to read the main parts of an Apache access log line and how to use `cat`, `grep`, and `less` to search it by hand.

I also learned that I do not have to memorize every event ID or do all analysis manually in a real environment, since a SIEM usually does much of this work. Even so, knowing how to read the raw logs helps me understand what the tools are showing me.

## Conclusion

This room gave me a first practical introduction to log analysis. I can now explain what logs are, name the main types, use Event Viewer to filter for specific Windows security events, and search an Apache access log from the command line. My experience so far is limited to these guided exercises. I have not worked with a real SIEM or analyzed firewall logs, and I still want to study the details of the access log fields and user agent strings further.
