# TryHackMe Shells Overview Notes

## Introduction

I completed the guided Shells Overview room on TryHackMe to learn how shells are used in cybersecurity, how remote command access works, and how listeners and payloads work together. The room covered command-line shells, reverse shells, bind shells, listener tools, shell payloads, and web shells. I also practiced obtaining a shell in the room's lab environment and retrieving the provided flag.

These notes record the concepts I studied and the actions described in my lab session. The payloads shown in the room are dual-use techniques. I treat them as material for understanding authorized security testing and for recognizing potentially dangerous behavior, not as permission to deploy them against systems without authorization.

## What a Shell Is

A shell is software that provides a way to interact with an operating system. A graphical shell provides a visual interface, while a command-line shell accepts commands as text. In cybersecurity, the term *shell* often refers to command-line access obtained on a remote system, particularly when that access was not intended by the system owner.

A shell does not automatically provide unrestricted control of a machine. The commands it can run depend on the account and permissions under which the shell process operates. An initial shell may run as a low-privilege service account, so it may not be able to read protected files, change system settings, or administer the machine. Privilege escalation refers to exploiting a weakness or misconfiguration to obtain higher permissions. It is a separate step from obtaining the initial shell.

After gaining access to one system, an attacker may attempt to reach other systems on the same network. This is called *pivoting*. A compromised machine may provide a route to systems that are not directly reachable from the attacker's original machine. Shell access can also be used to run commands, inspect files, gather information, maintain access, or transfer data. These are possible uses of shell access, not actions I performed during every part of this room.

## Reverse Shells and Bind Shells

A reverse shell and a bind shell differ mainly in which system initiates the network connection. In a reverse shell, the target initiates an outbound connection to a listener controlled by the operator. In a bind shell, a service on the target listens on a port, and the operator connects to that service.

Reverse shells are commonly demonstrated in security labs because outbound connections may be permitted even when unsolicited inbound connections to the target are blocked. This does not mean that a reverse shell is inherently stealthy or that a firewall will always allow it. Egress filtering, endpoint detection, network monitoring, and other controls can detect or block suspicious outbound connections. Using a familiar port such as TCP 443 does not make a connection safe or guarantee that it will blend in with legitimate HTTPS traffic. A raw Netcat connection on port 443 is not automatically HTTPS.

A reverse shell normally involves a listener waiting for an incoming connection and a payload that causes the target to connect back. In the lab, I used Netcat to listen on TCP port 444 and then submitted a reverse-shell payload through the provided exercise. The listener reported an incoming connection, and I was able to enter commands through the resulting shell. This was a guided lab activity on the supplied target.

A bind shell reverses the connection direction. The target runs a listener on a chosen port, and the operator connects to the target's address and port. This can be useful in a controlled lab when the target can accept inbound connections but cannot initiate an outbound connection. Whether the connection succeeds depends on routing, host firewalls, network firewalls, and the listener's permissions.

On Unix-like systems, ports below 1024 are traditionally privileged ports. Binding to one generally requires root privileges or an appropriate capability. Ports such as 4444 or 8080 are above that threshold, but their availability still depends on local policy and whether another process is already using the port.

## Netcat and Listener Tools

Netcat is a command-line networking utility that can read and write data over network connections. It is often used in training labs to create simple TCP listeners and test connections. In the room, the listener command followed this pattern:

```bash
nc -lvnp 444
```

Here, `nc` invokes Netcat. The `-l` option requests listening mode, `-v` enables verbose status output, `-n` disables name resolution, and `-p 444` specifies the local port. The exact supported options can vary between Netcat implementations, so I should check the installed tool's help or manual if a command behaves differently.

The room also introduced `rlwrap`, `ncat`, and `socat`. `rlwrap` can add readline-style command editing and history to interactive command-line programs that do not provide those features themselves. Ncat is a networking utility from the Nmap project and supports features that differ from traditional Netcat implementations, including optional SSL/TLS support. `socat` connects data streams and can work with network sockets, files, terminals, and other input/output endpoints. Its flexibility makes it useful for more complex networking tasks, but it also means its syntax requires careful reading.

A *socket* is a software endpoint used for communication. A network connection typically involves a socket at each end, identified by address and port information for IP networking. A listener is a process waiting for an incoming connection on a local endpoint; it is not itself the same thing as the entire connection.

## Standard Input, Output, Error, and File Descriptors

Shell payloads often redirect a program's input and output so that commands can be entered through a network connection and their results returned through that same connection. Understanding standard streams is therefore important.

Standard input, usually called `stdin`, is file descriptor `0`. It is where a process receives input. Standard output, or `stdout`, is file descriptor `1` and carries normal command output. Standard error, or `stderr`, is file descriptor `2` and carries diagnostic messages and errors. In a terminal session, these streams are usually connected to the terminal, but a program can redirect them elsewhere.

The `>` operator redirects standard output to a destination. The `<` operator redirects standard input from a source. The expression `2>&1` redirects standard error to the same destination currently used by standard output. The order of redirections matters because the shell processes them from left to right. These operators do not mean that every output is always sent to the same place; the result depends on the exact command and the order of its redirections.

A *file descriptor* is a small integer used by a process to refer to an open input/output resource. Descriptors `0`, `1`, and `2` have the conventional meanings described above. Programs can open additional descriptors for files or sockets. Some payloads duplicate descriptors so that a shell's input, output, and error streams use a network socket instead of the terminal.

A named pipe, also called a FIFO, is a communication mechanism that lets processes exchange data through a filesystem name. It is not an ordinary file that permanently stores all written content. Processes can use it to pass data between each other, and reads and writes can block while waiting for the other side. The room used a FIFO-based example to illustrate how input from a connection can be passed to a shell and how the shell's output can be returned. The command involved several shell features at once, so I focused on the underlying pattern: establish a connection, connect the shell's standard streams to it, and keep input and output flowing between the two endpoints.

## Shell Payloads

A payload is the code or command that performs the intended action after it runs. In the shell examples, the payload is responsible for creating or using a connection and attaching a command interpreter to it. The listener and payload serve different roles: the listener waits for a connection, while the payload initiates or handles the other end.

The room demonstrated that similar shell behavior can be implemented in different languages and tools. Bash examples used shell redirection and, in some cases, `/dev/tcp` support to create a network connection. Other Bash examples used `exec` and additional file descriptors to refer to a connection and redirect the shell's streams. These examples are useful for learning how Unix file descriptors and redirections fit together, but their exact behavior depends on the shell and environment.

The PHP examples used functions such as `exec`, `shell_exec`, `system`, `passthru`, and `popen` to invoke operating-system commands or interact with processes. These functions do not all behave identically: they differ in how they return output, handle streams, and expose process input/output. Their presence in a web application is not automatically proof of a vulnerability, but using them with untrusted input can create command-injection risks. Applications should avoid passing untrusted data to operating-system command interpreters and should use safer APIs and strict input handling where possible.

The Python examples used the `socket` module to create network sockets, operating-system interfaces to work with file descriptors, and pseudo-terminal functionality to provide an interactive shell-like experience. The examples also illustrated environment variables, which processes can inherit from their parent environment. I studied these payloads to recognize the recurring structure rather than to claim that I independently wrote or developed them.

The room also showed that other utilities and scripting languages can be adapted for similar purposes. A Telnet-based example used a temporary named pipe to connect input and output, while an AWK example demonstrated that a general-purpose text-processing language can be misused to run commands and communicate over a socket. BusyBox was mentioned because it bundles many common Unix utilities and is often found on small or embedded systems. Availability of a utility does not guarantee that a particular payload will work; the target's installed tools, shell implementation, permissions, and network controls all matter.

## Web Shells

A web shell is a script placed on a web server that allows requests to trigger server-side actions, often including operating-system command execution. PHP and ASP.NET are examples of technologies that may be used to create web shells, depending on the server environment. A web shell can be particularly dangerous because an attacker may be able to invoke it through ordinary HTTP requests if the file is reachable and the server executes it.

A simple PHP web shell may read a command from a request parameter and pass it to a command-execution function. For example, a script that reads a value from `$_GET` and passes it to `system()` can allow a request parameter to influence an operating-system command. The core risk is not merely that the application uses PHP. The risk is that externally controlled input reaches a command-execution function without safe design and validation.

Uploading a script to a server is not the same as executing it. For a web shell to work, the attacker generally needs a way to place the file in a location where the server can access it, and the server must be configured to execute that file type in that location. An unrestricted file upload vulnerability occurs when an application fails to enforce suitable restrictions on uploaded files. Depending on the application and server configuration, this can allow an attacker to upload a script that later executes. Storing user uploads outside executable web directories, enforcing allowlists and content checks, applying least privilege, and preventing uploaded content from being interpreted as server-side code are important defensive measures.

A web shell may provide a basic command parameter and return the output in the HTTP response, or it may offer a more complete interface with file-management and other features. The room referenced examples of PHP web shells, including Pony, b374k, and C99. I learned about their existence and general purpose; I did not document independently testing each of those tools.

## Practical Lab Work

The room provided two practical exercises. In the reverse-shell exercise, I started a Netcat listener on my attack-box environment, changed the example payload to use the attack-box IP address and selected port, and submitted it through the lab page. The listener reported a connection and presented a shell. I used `ls` to inspect the current directory and `whoami` to identify the account running the shell. The output showed `www-data`, which is commonly used as a web-server service account on Linux systems. This indicated that the shell was running with the privileges of that account rather than automatically providing root access.

I then navigated to the root directory as instructed by the lab and read the provided flag file with `cat`. The exercise demonstrated the relationship between the listener, the payload, the remote shell, and the commands entered after the connection was established. It also reinforced that the shell's effective permissions determine which files and operations are accessible.

In the web-shell exercise, I created a PHP file named `shell.php` containing the room's example web-shell code, started a listener, and uploaded the file through the supplied web application. I then navigated to the uploaded script and supplied a command through its request parameter to make the script run the requested command. I used the resulting access to locate and read the lab's flag file. This was a guided exercise on the TryHackMe target, not an assessment of a real production web server.

The practical work illustrated two different paths to remote command execution. The reverse shell created a connection back to the listener and provided an interactive command channel. The web shell accepted an HTTP request and executed a command on the server in response. Both depended on the target environment and the permissions of the process executing the command.

## Security Significance and Defensive Considerations

Shell access is significant because it can turn a limited vulnerability into direct interaction with the operating system. The actual impact depends on the account privileges, accessible files, network reachability, and security controls. A low-privilege shell can still expose application data or provide a starting point for further investigation, but it should not be described as unrestricted control unless that level of access has actually been demonstrated.

From a defensive perspective, unexpected command interpreters, suspicious child processes launched by web-server services, unusual outbound connections, unexpected listeners, and newly uploaded executable scripts can all warrant investigation. These signals need to be evaluated in context because legitimate administration and application behavior can produce similar activity. Restricting unnecessary outbound traffic, monitoring process and network activity, keeping web applications patched, limiting service-account permissions, and separating user uploads from executable content can reduce the likelihood or impact of shell-based compromise.

The use of a commonly known port does not make malicious activity invisible. Network defenders should evaluate the protocol and behavior of a connection, not just its port number. For example, a raw TCP shell on port 443 does not behave like a normal TLS-protected HTTPS session simply because it uses the same port.

## What I Learned

I learned that a shell is a way to interact with an operating system and that remote shell access is constrained by the permissions of the process that provides it. I can now explain the difference between reverse shells and bind shells in terms of which side initiates the connection, and I understand why a listener and a payload are separate parts of the process.

I also learned why standard input, standard output, standard error, redirection operators, file descriptors, sockets, and named pipes appear repeatedly in shell payloads. These concepts explain how commands can receive input through a network connection and return output over that connection. The examples across Bash, PHP, and Python showed different ways to implement the same broad behavior, although I would need more practice to read every payload confidently without guidance.

The practical exercises let me use a listener, interact with a shell in the supplied lab, identify the account running the shell, and retrieve the lab's flag. I also uploaded and accessed a basic PHP web shell in the lab. These were guided exercises, and I do not treat them as professional penetration-testing experience or as evidence that I can independently develop complex payloads.

## Conclusion

The main lesson from the Shells Overview room is that remote shell access depends on more than a single command. It involves a way to execute code on the target, a connection between two endpoints, and correct handling of the shell's input and output. Reverse shells, bind shells, and web shells differ in how that access is established, but each can expose operating-system functionality through a process running with specific permissions. Understanding these mechanics also helps me recognize behaviors that defenders may need to investigate.
