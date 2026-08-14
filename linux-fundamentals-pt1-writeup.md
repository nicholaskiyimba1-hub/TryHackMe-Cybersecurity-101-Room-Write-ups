# TryHackMe Write-up: Linux Fundamentals Part 1

**Write-up by: Nicholas Kiyimba**

## Introduction

This is my write-up for the Linux Fundamentals Part 1 room on TryHackMe. This room introduces the basics of using the Linux terminal. I am writing this to document what I learned and to keep a personal record of my progress as I work through TryHackMe.

Linux is used in more places than most people realize. It runs websites, car entertainment systems, point of sale machines in shops, traffic light controllers, phones, and many other devices. Even though it looks intimidating at first because it relies on typed commands instead of a mouse, it becomes much easier with practice.

## Task 1: Setting Up My Environment

Before doing anything else, I started the lab machine provided in the room. This gave me a Linux virtual machine that I could access directly from my browser, along with its IP address and a timer showing how long I had left with it. I made sure to terminate the machine once I was done with the room, since leaving it running is not good practice.

## Task 2: Talking to Linux

This task introduced me to the terminal itself, which is the main way I will be interacting with Linux going forward. Instead of clicking with a mouse, I type commands and the system responds with output. It genuinely feels like a conversation with the machine.

The two commands I practiced here were:

- `whoami` – tells me who I am logged in as on the system
- `echo` – prints out text that I give it, for example `echo "hello world"`

Knowing who I am on a system is important in cyber security because my username determines what I am allowed to do. I will often need to switch between different users, so checking my identity with `whoami` is something I will use constantly.

## Task 3: Finding My Way Around

This task covered the four commands I will use for basic navigation:

- `ls` – lists what is inside the current folder
- `cd` – changes to a different folder
- `cat` – shows the contents of a file
- `pwd` – prints my current location in the system

I used `ls` to see what was in my home folder, then used `cd` to move into a folder and `ls` again to see what was inside it. I used `cat` to read the contents of a file I found, and used `pwd` whenever I needed to confirm exactly where I was in the file system.

Folders and files on this machine are shown in different colours, which makes them easy to tell apart at a glance. I found this navigation method awkward at first since I am used to a mouse, but I can already see how it would be much faster once I get comfortable with it.

## Task 4: Letting the Machine Search

Rather than reading through a file line by line, Linux gives us tools to search automatically:

- `find` – searches for files by their name, for example `find -name passwords.txt`
- `grep` – searches inside a file for specific text, for example `grep "password123" passwords.txt`

For this task, there was a web server log file called `access.log` in my home folder that was hundreds of lines long. I navigated back to my home folder and used `grep` to search the log for a specific keyword, which located the flag hidden inside the file without me having to scroll through it manually.

This showed me how useful `grep` and `find` are. Real systems can have huge log files, and searching manually would take far too long. These commands let me pull out exactly what I need in seconds.

## Task 5: Shell Operators

The last task covered special characters that let me combine commands or control where their output goes:

- `&` – runs a command in the background without waiting for it to finish
- `&&` – runs a second command only after the first one finishes successfully
- `>` – sends a command's output into a file, overwriting whatever was already there
- `>>` – sends a command's output into a file, but adds it to the end instead of overwriting

I practiced this by using `echo` together with `>` to create a file containing some text, then using `>>` to add more text to that same file without deleting what was already there. I confirmed the results each time with `cat`.

This task tied everything together for me. Being able to combine commands and control where output goes is something I will be using constantly going forward, whether I am saving results from a scan or building up a file step by step.

## What I Learned

By the end of this room, I was comfortable with:

- Checking my identity on a Linux system
- Navigating folders and reading files from the terminal
- Searching for files and searching inside files
- Combining commands and redirecting their output

This is a solid foundation for the more advanced Linux rooms ahead. I completed this room myself as part of my personal TryHackMe learning path.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
