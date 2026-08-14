# TryHackMe Write-up: Linux Fundamentals Part 2

**Write-up by: Nicholas Kiyimba**

## Introduction

This is my write-up for the Linux Fundamentals Part 2 room on TryHackMe. This room builds on what I covered in Part 1, but instead of using the in-browser terminal, it moves on to connecting to a remote Linux machine using SSH. It also covers flags and switches, file permissions, and some of the important directories on a Linux system.

## Task 2: Accessing My Linux Machine Using SSH

For this task I deployed two machines: my own Linux lab machine, and the TryHackMe AttackBox, which is a Ubuntu machine hosted in the cloud that I access through the browser.

I learned that SSH (Secure Shell) is a protocol used to connect to and control another machine remotely. Anything I type is encrypted before it travels over the network, and it is only decrypted once it reaches the remote machine. This means someone intercepting the traffic cannot read what I am sending.

To connect, I only needed two things: the IP address of the remote machine, and valid login credentials. From the AttackBox terminal, I ran:

```
ssh tryhackme@MACHINE_IP
```

I replaced `MACHINE_IP` with the actual IP address shown for my lab machine, then logged in with the username and password provided for the room. One thing I noted is that when typing a password into an SSH prompt, nothing appears on screen at all, not even asterisks. This felt strange at first, but it is normal behaviour, so I just typed the password and pressed enter. Once connected, any command I ran was now executing on the remote machine, not on the AttackBox itself.

## Task 3: Flags and Switches

Most Linux commands accept extra arguments called flags or switches, which change how the command behaves. These are usually written with a hyphen before them.

A good example is `ls`. On its own, `ls` just lists the files and folders in the current directory, but it does not show hidden files. Hidden files and folders in Linux start with a dot, for example `.hiddenfolder`. Adding the `-a` flag (short for `--all`) reveals these hidden items:

```
ls -a
```

I also learned that most commands support a `--help` option, which prints a short summary of the flags that command accepts. For a more detailed explanation, Linux provides manual pages, accessed with the `man` command followed by the command name, for example:

```
man ls
```

This opens the full documentation for `ls` directly in the terminal. I practiced navigating this manual page using the arrow keys to scroll down, and found the flag that displays file sizes in a "human-readable" format (for example showing `4K` instead of `4096`), which is the `-h` flag.

## Task 4: File System Interaction Continued

This task built on the navigation commands from Part 1, giving me more practice moving around the filesystem and working with files and folders on the actual remote machine rather than the in-browser one.

## Task 5: Permissions 101

This task explained how Linux controls who can access a file or folder, which is something I will need to understand well for cyber security work.

Running `ls -lh` on a file shows a permissions string made up of ten characters, but the important part is the first nine, split into three groups of three:

- Owner permissions
- Group permissions
- Permissions for everyone else (others)

Each group can have:

- `r` = read
- `w` = write
- `x` = execute

For example, `rwxr-xr-x` means the owner can read, write, and execute the file, while the group and everyone else can only read and execute it.

I also learned that these permissions can be written as numbers instead of letters, since each permission has a value:

- Read = 4
- Write = 2
- Execute = 1

Adding these values together for each group gives a three-digit number. So `rwxrwxrwx` becomes `777`, and `rwxr-xr-x` becomes `755`. This numeric format is used with commands like `chmod`, for example `chmod 750 file.txt` gives the owner full access, the group read and execute access, and no access at all to everyone else.

The task also covered switching between users with the `su` command. To switch to another user I need to know their username and password. Using `su -l username` (with the `-l` or `--login` flag) starts a fresh login session as that user, dropping me into their home directory, which behaves more like actually logging in as them rather than just borrowing their permissions temporarily.

On the deployed machine, I used `su` to switch to "user2" using the provided password, then used `cat` to read a file called "important" that belonged to that user, which contained the flag for this task.

## Task 6: Common Directories

This task walked through some of the key root directories on a Linux system and what they are used for:

- **/etc** – short for "etcetera", this is where system configuration files are kept. It contains files like `sudoers` (which lists who can run commands as root), and `passwd` and `shadow`, which store user account and password information in encrypted form.
- **/var** – short for "variable data", this stores data that changes often, such as log files (`/var/log`) written by running services and applications.
- **/root** – this is the home directory for the root user specifically, separate from the regular `/home` directory used by other accounts.
- **/tmp** – short for "temporary", this folder stores short-term data. Its contents are cleared out when the machine restarts, similar to how RAM works. Any user can write to `/tmp` by default, which makes it a common location to drop scripts or tools once access to a machine has been gained.

## What I Learned

By the end of this room, I was comfortable with:

- Connecting to a remote Linux machine securely using SSH
- Using flags and switches to change how commands behave, and using `man` pages to look up what flags are available
- Understanding and reading Linux file permissions in both symbolic and numeric format
- Switching between user accounts with `su`
- Knowing what the key system directories (`/etc`, `/var`, `/root`, `/tmp`) are used for

This room felt more theory-heavy than Part 1, but it gave me a much better understanding of how a real Linux system is structured and how access control works, which I know will come up constantly in cyber security work. I completed this room myself as part of my personal TryHackMe learning path.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
