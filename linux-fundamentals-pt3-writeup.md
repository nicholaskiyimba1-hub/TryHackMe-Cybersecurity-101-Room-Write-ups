# TryHackMe Write-up: Linux Fundamentals Part 3

**Write-up by: Nicholas Kiyimba**

## Introduction

This is my write-up for Linux Fundamentals Part 3, the final room in the Linux Fundamentals series on TryHackMe. This room moved past the basics and introduced some day-to-day utilities I will actually be using, including text editors, file transfers, process management, and how to automate and maintain a Linux system.

## Task 2: Deploying My Linux Machine

I started both my lab machine and the TryHackMe AttackBox, then connected to my lab machine from the AttackBox using SSH with the provided credentials:

```
ssh tryhackme@MACHINE_IP
```

## Task 3: Terminal Text Editors

Up to this point I had only been writing text into files using `echo` combined with the `>` and `>>` redirectors, which is not practical for anything longer than a line or two. This task introduced proper terminal text editors.

**Nano** is the beginner-friendly option. To create or edit a file, I just run:

```
nano filename
```

This opens the file directly in the terminal, where I can type or edit text and move around with the arrow keys. Nano shows its available shortcuts along the bottom of the screen, using the `Ctrl` key (shown as `^`) combined with a letter. For example, `Ctrl + X` exits the editor. It supports basics like searching text, copying and pasting, and jumping to a specific line number.

**VIM** was introduced as a more advanced alternative. It takes longer to learn, but it is far more powerful; it is customizable, supports syntax highlighting (useful for writing code), and is available on almost every Linux system, even ones where Nano is not installed. TryHackMe has a separate room dedicated to VIM if I want to go deeper into it later.

For this task, I used Nano to edit a file called "task3" in the home directory of the "tryhackme" user, which revealed the flag for this task.

## Task 4: General/Useful Utilities

This task covered practical ways of moving files onto and off of a Linux machine:

**wget** downloads a file from the web over HTTP, the same way a browser would, by just providing the file's address:

```
wget https://example.com/myfile.txt
```

**SCP (Secure Copy)** transfers files between two computers using the SSH protocol, which handles both authentication and encryption. It works on a SOURCE and DESTINATION model. To copy a file from my machine to a remote machine:

```
scp important.txt ubuntu@192.168.1.30:/home/ubuntu/transferred.txt
```

And to copy a file from a remote machine down to my own, the source and destination are simply reversed:

```
scp ubuntu@192.168.1.30:/home/ubuntu/documents.txt notes.txt
```

**Serving files with Python** was the last method covered. Ubuntu machines come with Python 3 already installed, which includes a lightweight module called HTTPServer. Running:

```
python3 -m http.server
```

turns the current directory into a simple web server, by default on port 8000. From another terminal (since the server occupies the one it's running in), I can then use `wget` to pull a file from that server:

```
wget http://MACHINE_IP:8000/myfile
```

For this task, I started the Python web server in the tryhackme user's home directory on the lab machine, then used `wget` from the AttackBox in a separate terminal to download a hidden file named `.flag.txt`, which contained the flag. I stopped the server afterward with `Ctrl + C`.

## Task 5: Processes 101

This task covered what processes are and how Linux manages them. Every running program is a process, and each one is given a Process ID (PID) in the order it started, so a process starting after PID 300 would be assigned PID 301.

To view processes, I used:

- `ps` – shows the processes running in my current session
- `ps aux` – shows all processes on the system, including ones run by other users and system processes that aren't tied to a session
- `top` – shows real-time, constantly refreshing statistics about running processes and system resource usage

To manage processes, I can send them signals using the `kill` command along with a PID, for example `kill 1337`. Some important signals are:

- **SIGTERM** – asks the process to end cleanly, giving it a chance to do cleanup first
- **SIGKILL** – forces the process to end immediately with no cleanup
- **SIGSTOP** – pauses/suspends a process

I also learned about namespaces, which is how the operating system divides up resources like CPU and RAM between processes, and helps keep processes isolated from each other for security. The very first process on boot has PID 0 and is the system's init process (systemd on Ubuntu). Every other process starts as a child of systemd.

For managing services that need to run automatically, `systemctl` is used to interact with systemd. It supports five main actions: start, stop, enable, disable, and status. For example, to stop a service called "myservice":

```
systemctl stop myservice
```

And to make sure that same service starts automatically on boot:

```
systemctl enable myservice
```

Finally, this task covered backgrounding and foregrounding processes. Adding `&` to the end of a command runs it in the background instead of waiting for it to finish before I can type another command. I can also background an already-running process using `Ctrl + Z`. To bring a backgrounded process back into focus, I use the `fg` command.

## Task 6: Maintaining Your System - Automation

This task introduced cron and crontabs, which are used to schedule tasks to run automatically, such as backups or launching a program on a schedule.

A crontab entry needs six values in this order: minute, hour, day of month, month, day of week, and the command to run. For example, backing up a Documents folder every 12 hours would look like:

```
0 */12 * * * cp -R /home/cmnatic/Documents /var/backups/
```

The asterisk (`*`) acts as a wildcard, meaning "any value" for that field. Crontabs can be edited using `crontab -e`, which opens the crontab file in a text editor like Nano. On the deployed instance, I checked the existing crontab to see exactly when the scheduled job was set to run.

## Task 7: Maintaining Your System - Package Management

This task explained how software gets onto a Linux system through package repositories. Developers submit their software to an "apt" repository, and once approved, it becomes available for anyone to install. Ubuntu vendors maintain their own default repositories, but community or third-party repositories can also be added to extend what's available.

Software is normally managed using the `apt` command, which is part of a larger suite of tools for managing packages and their sources. Adding a new repository involves trusting a GPG key from the developer first, which acts as a safety check to confirm that the software genuinely comes from them and has not been tampered with. Once trusted, the repository is added as a file under `/etc/apt/sources.list.d/`, followed by running `apt update` to refresh the package lists, and then `apt install package-name` to install the software.

Removing a repository and its software works the same way in reverse, using `add-apt-repository --remove` or manually deleting the repository file, followed by `apt remove package-name`.

Since the TryHackMe lab machines don't have internet access, this task was reading-only rather than hands-on, but I now understand the overall process of safely adding and removing software sources on Ubuntu.

## Task 8: Maintaining Your System - Logs

This task expanded on the log files briefly mentioned back in Part 1. Logs are stored in `/var/log`, and the operating system automatically manages and rotates these logs over time so they don't grow indefinitely.

I looked at logs from a few different services as examples:

- **Apache2** – a web server, which logs every request it receives
- **fail2ban** – a service that monitors and blocks repeated failed login attempts, such as brute-force attacks
- **UFW** – the Uncomplicated Firewall, which logs allowed and blocked connections

For a web server specifically, the two log files worth knowing are the access log (records every request made to the site) and the error log (records problems the server encountered). I examined the Apache2 access log on the deployed machine to identify the IP address of a visitor and which file they had accessed, which is exactly the kind of information that would help in diagnosing an issue or investigating suspicious activity on a real system.

## What I Learned

By the end of this room, and the Linux Fundamentals series as a whole, I was comfortable with:

- Writing and editing files properly using Nano, and knowing VIM exists as a more advanced option
- Downloading and transferring files using `wget` and `scp`, and serving files with Python's built-in web server
- Viewing and managing running processes, including backgrounding, foregrounding, and killing processes cleanly
- Scheduling automated tasks using cron and crontabs
- Understanding how software packages and repositories work on Ubuntu
- Knowing where to find and how to read system and application logs

This wraps up the Linux Fundamentals series for me. I now have a solid working knowledge of the Linux terminal, from basic navigation all the way through to automation and system maintenance, which I'll build on in more specialized rooms going forward. I completed this room myself as part of my personal TryHackMe learning path.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
