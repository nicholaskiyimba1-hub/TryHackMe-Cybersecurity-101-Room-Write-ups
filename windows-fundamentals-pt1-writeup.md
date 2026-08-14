# TryHackMe Write-up: Windows Fundamentals Part 1

**Write-up by: Nicholas Kiyimba**

## Introduction

This is my write-up for the Windows Fundamentals Part 1 room on TryHackMe. After spending time learning the basics of Linux, this room shifted my focus to the Windows operating system, covering its history, the desktop interface, the file system, user accounts, and some of the built-in tools used to manage a Windows machine.

## Task 1: Windows Editions

This task gave a short history of Windows. It has been around since 1985 and is the most widely used operating system in both homes and companies, which is exactly why it is such a common target for hackers and malware writers.

I learned about how Windows has evolved over the years: Windows XP was popular for a long time, then Windows Vista was released as a major overhaul but was not well received. Windows 7 followed and was much more successful, then Windows 8.x came and went quickly, and eventually Windows 10 and now Windows 11 became the current versions for desktop computers. On the server side, the current version is Windows Server 2025.

Windows 11 comes in two editions, Home and Pro, and one of the differences is that Pro supports BitLocker device encryption, which is not available on the Home edition.

## Task 2: The Desktop GUI

This task covered the Windows Desktop, which is the graphical interface that appears once you log into a Windows 10 machine. Before reaching the desktop, a user has to pass the login screen by entering valid account credentials.

I went through the main components that make up the desktop environment:

- **The Desktop** – where shortcuts to programs, folders, and files are placed for quick access. Right-clicking the desktop brings up a menu to change icon size, arrangement, and to create new items. Display and personalization settings (like screen resolution and wallpaper) can also be changed from here.
- **The Start Menu** – opened by clicking the Windows logo. It is split into sections: account shortcuts (like locking the screen or signing out), a list of recently added and all installed apps in alphabetical order, and tiles for pinned apps and programs on the right side.
- **The Search Box (Cortana)** – used to search for apps, files, and settings.
- **Task View** – shows open windows and virtual desktops.
- **The Taskbar** – shows any open apps, folders, or files, and provides a preview thumbnail when hovering over an icon. Right-clicking the taskbar lets me enable or disable different taskbar components.
- **Toolbars** – additional shortcuts that can be enabled on the taskbar.
- **The Notification Area** – usually at the bottom right, showing the date, time, and icons like volume and network status. Icons here can be added or removed through taskbar settings.

## Task 3: Introduction to Windows (Lab Setup)

For this task I started the lab machine, which is a Windows Server 2019 Standard machine. I connected to it using the AttackBox and Remote Desktop, logging in as the administrator account with the credentials provided in the room. I accepted the certificate warning when prompted, which then let me into the remote system.

## Task 4: The File System

This task introduced NTFS (New Technology File System), which is the file system used by modern Windows installations. Before NTFS, Windows used FAT16/FAT32 and HPFS. FAT partitions are still commonly found today, but usually on USB drives and memory cards rather than on the main drive of a Windows computer.

NTFS is a journaling file system, meaning it can repair files and folders automatically after a failure using information stored in a log. This is something FAT is not able to do. NTFS also improved on older file systems by:

- Supporting files larger than 4GB
- Allowing specific permissions to be set on folders and files
- Supporting folder and file compression
- Supporting encryption (Encrypting File System)

NTFS permissions include Full control, Modify, Read & Execute, List folder contents, Read, and Write. To view these permissions, I right-click a file or folder, go to Properties, then the Security tab, and select a user or group to see their assigned permissions.

I also learned about Alternate Data Streams (ADS), a feature specific to NTFS. Every file has at least one data stream by default, but ADS allows a file to hold more than one stream of data. Windows Explorer does not show these extra streams by default, though PowerShell and some third-party tools can reveal them. ADS is often used to mark files downloaded from the internet, but it has also been used by malware to hide data, which makes it worth knowing about from a security standpoint.

## Task 5: The Windows and System32 Folders

The Windows folder (`C:\Windows`) is the folder that holds the Windows operating system itself. It does not always have to be on the C drive, which is why Windows uses a system environment variable, `%windir%`, to refer to it regardless of where it actually is.

Inside the Windows folder is the System32 folder, which holds files that are critical to the operating system. I learned to be very careful around this folder, since deleting files here by mistake can stop Windows from working properly. Many of the built-in tools covered later in the Windows Fundamentals series are actually stored inside System32.

## Task 6: User Accounts, Profiles, and Permissions

This task covered the two main types of local Windows accounts:

- **Administrator** – can make system-level changes such as adding or removing users, modifying groups, and changing system settings.
- **Standard User** – can only make changes within their own files and folders, and cannot perform system-level changes like installing programs.

I checked the existing accounts on the system through Start Menu search for "Other User", which opens Settings > Other users. Since I was logged in as an administrator, I had the option to add someone else to the PC, an option a standard user would not see.

Every user account gets its own profile folder under `C:\Users`, for example `C:\Users\Max`. This profile is created the first time that user logs in, and includes standard folders like Desktop, Documents, Downloads, Music, and Pictures.

I also explored Local Users and Groups by opening the Run dialog (right-click Start Menu) and typing `lusrmgr.msc`. This tool shows two folders, Users and Groups, and lets me see which groups a user belongs to, along with a description of each group. A user inherits the permissions of any group they are added to, and can belong to more than one group. I used this tool to check the other user account on the machine, which groups it belonged to, and confirmed that the built-in account for guest access is the Guest account.

## Task 7: User Account Control (UAC)

This task explained User Account Control, or UAC, a Windows security feature first introduced in Windows Vista. Most home users are logged in as administrators, but running everyday tasks like browsing the internet with full administrator privileges all the time increases the risk that malware could make unauthorized changes to the system.

UAC works by keeping even an administrator's session running with standard privileges most of the time. When an action needs higher-level privileges, Windows prompts the user to confirm before allowing it to proceed. I noticed that programs requiring elevated privileges show a small shield icon, which is a visual warning that a UAC prompt will appear.

I tested this by logging in as the standard user and trying to install a program. The UAC prompt appeared asking for the administrator's password, and if no password is entered after a short while, the prompt disappears and the installation does not proceed. This means UAC does not apply by default to the built-in local administrator account, but it does help protect systems where users are logged in with administrator-type accounts.

## Task 8: Settings and the Control Panel

This task compared the two main places to change system settings on Windows: the Settings menu and the Control Panel.

The Control Panel has traditionally been the place to make changes like adding a printer or uninstalling software. The Settings menu was introduced in Windows 8 for touchscreen devices and has since become the main place most users go to change system settings in Windows 10.

I used the Control Panel's Programs and Features section to see a list of all installed applications along with their names, publishers, and versions, which is a useful way to check what software is present on a machine. I also found that the two menus are connected in places. For example, going to Network & Internet in Settings and then Change adapter options actually opens a Control Panel window. When unsure where a setting lives, searching for it directly from the Start Menu usually finds the right place either way.

## Task 9: The Task Manager

This task introduced the Task Manager, which shows the applications and processes currently running on the system, along with performance information such as CPU and memory usage.

I opened it by right-clicking the taskbar, and by default it opens in a simple view showing only running apps. Clicking "More details" expands it to show far more information, including processes, performance graphs, and resource usage. The keyboard shortcut to open Task Manager directly is Ctrl + Shift + Esc.

## What I Learned

By the end of this room, I had a much better general understanding of:

- The history and evolution of Windows editions
- The layout and components of the Windows desktop
- How NTFS works and what makes it different from older file systems
- The purpose of the Windows and System32 folders
- The difference between administrator and standard user accounts, and how user profiles and groups work
- How User Account Control protects the system from unauthorized changes
- Where to go for basic and advanced system settings
- How to use Task Manager to monitor running processes and performance

This room was a good foundation for understanding how a Windows system is put together, which I know will be useful as I move into more security-focused Windows rooms. I completed this room myself as part of my personal TryHackMe learning path.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
