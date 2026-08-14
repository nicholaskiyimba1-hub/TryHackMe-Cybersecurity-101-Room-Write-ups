# Linux Shells — TryHackMe Write-up

## Overview

In this room, I learned the fundamentals of Linux shells and Bash scripting. The room introduced how users interact with Linux through a shell, the different types of Linux shells available, the basics of shell scripting, and how these concepts are applied to automate tasks. I also completed a practical exercise that involved debugging and executing a Bash script to locate a hidden flag.

---

# Learning Objectives

By the end of this room, I was able to:

- Understand what a Linux shell is and its role in an operating system.
- Differentiate between Bash, Fish, and Zsh.
- Navigate the Linux command line using essential commands.
- Understand the basic structure of a Bash script.
- Use variables, loops, conditional statements, and comments in Bash scripts.
- Execute Bash scripts and manage execution permissions.
- Read, debug, and modify an existing Bash script.
- Search files using `grep`.
- Work with Linux file paths and directories.

---

# Key Concepts Learned

## 1. Linux Shell

A shell is a command interpreter that acts as the interface between the user and the Linux operating system. It receives commands from the user, interprets them, and communicates with the Linux kernel to perform the requested actions.

Instead of interacting directly with the operating system, users interact with the shell.

---

## 2. Essential Linux Commands

During this room, I reviewed several important Linux commands.

| Command | Purpose |
|---------|---------|
| `pwd` | Displays the current working directory. |
| `cd` | Changes the current directory. |
| `ls` | Lists files and directories. |
| `cat` | Displays the contents of a file. |
| `grep` | Searches for text or patterns inside files. |
| `history` | Displays previously executed commands. |
| `echo` | Prints text or variable values to the terminal. |

---

## 3. Types of Linux Shells

### Bash (Bourne Again Shell)

- Default shell on most Linux distributions.
- Excellent scripting support.
- Command history.
- Tab completion.
- Most commonly used in cybersecurity and system administration.

### Fish (Friendly Interactive Shell)

- Beginner-friendly.
- Built-in syntax highlighting.
- Auto spell correction.
- Interactive command suggestions.

### Zsh (Z Shell)

- Advanced tab completion.
- Auto correction.
- Extensive customization.
- Supports plugins such as Oh My Zsh.

---

# Shell Scripting

A shell script is a text file containing multiple Linux commands that are executed sequentially.

Instead of repeatedly typing the same commands, they can be automated using a script.

Example:

```bash
#!/bin/bash

echo "Hello World"
```

---

# Shebang

Every Bash script begins with:

```bash
#!/bin/bash
```

This tells Linux to execute the script using the Bash interpreter.

---

# Variables

Variables store values for later use.

Example:

```bash
name="Nicholas"

echo $name
```

Output:

```
Nicholas
```

Variables make scripts reusable and easier to maintain.

---

# User Input

The `read` command collects input from the user.

Example:

```bash
echo "Enter your name:"
read name

echo "Welcome $name"
```

---

# Execution Permissions

A newly created script is not executable by default.

Execution permission is granted using:

```bash
chmod +x script.sh
```

The script is then executed using:

```bash
./script.sh
```

The `./` tells Bash to execute the script from the current directory.

---

# Loops

Loops execute a block of code repeatedly.

Example:

```bash
for i in {1..5}
do
    echo $i
done
```

Output:

```
1
2
3
4
5
```

---

# Conditional Statements

Conditional statements execute code depending on whether a condition is true.

Example:

```bash
if [ "$name" = "John" ]; then
    echo "Access Granted"
else
    echo "Access Denied"
fi
```

---

# Comments

Comments improve code readability and are ignored during execution.

Example:

```bash
# This is a comment
```

---

# Practical Exercise

## Objective

A Bash script was provided to search for a specific keyword inside all `.log` files located in `/var/log`.

Initially, the script failed to locate the files because the loop contained an incorrect path.

Original loop:

```bash
for file in " "/*.log; do
```

The loop attempted to search an invalid location.

The corrected version:

```bash
for file in "$directory"/*.log; do
```

Since:

```bash
directory="/var/log"
```

Bash expanded the loop to:

```bash
for file in /var/log/*.log
```

allowing the script to iterate through every log file in the target directory.

---

## Running the Script

Become the root user:

```bash
sudo su
```

Grant execution permission:

```bash
chmod +x flag_hunt.sh
```

Execute the script:

```bash
./flag_hunt.sh
```

Output:

```text
Flag search in directory: /var/log in progress...
Flag found in: authentication.log
```

Display the contents of the identified log file:

```bash
cat /var/log/authentication.log
```

Output:

```text
the cat is sleeping under the table
thm-flag01-script
```

---

# Challenges Encountered

While completing the practical exercise, I initially ran:

```bash
cat /var/log authentication.log
```

This produced an error because:

- `/var/log` is a directory.
- `authentication.log` was interpreted as a file in the current directory (`/home/user`).

The correct command was:

```bash
cat /var/log/authentication.log
```

This reinforced the importance of using the correct absolute file path.

---

# Key Takeaways

- A shell acts as the communication layer between the user and Linux.
- Bash is the standard shell used on most Linux systems and is heavily used in cybersecurity.
- Shell scripts automate repetitive tasks.
- Variables store reusable information.
- Loops reduce repetitive code.
- Conditional statements allow scripts to make decisions.
- `chmod +x` grants execution permission.
- `./script.sh` executes a script located in the current directory.
- `grep` is an essential command for searching text inside files.
- Reading error messages carefully is often enough to identify and fix problems.
- Understanding file paths is critical when working in Linux.

---

# Skills Gained

- Linux command-line navigation
- Bash shell fundamentals
- Bash scripting basics
- Variables and user input
- Loops and conditional statements
- Script execution and permissions
- Linux file path management
- Basic script debugging
- Text searching with `grep`
- Practical troubleshooting

---

## Conclusion

This room strengthened my understanding of Linux shells and introduced me to the foundations of Bash scripting. Beyond learning the syntax, I gained practical experience reading, modifying, and debugging an existing script, using Linux commands to investigate files, and solving real-world problems through systematic troubleshooting. These are fundamental skills that will continue to support my journey in cybersecurity and penetration testing.