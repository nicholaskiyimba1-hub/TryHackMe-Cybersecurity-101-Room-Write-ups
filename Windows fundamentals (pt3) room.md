# Windows Fundamentals (Pt3)

**Write-up by: Nicholas Kiyimba**

## Introduction

This room continues on from Windows Fundamentals 1 and 2, which covered the desktop environment, UAC, the Control Panel, Settings, Task Manager, System Configuration, Computer Management, and Resource Monitor. This room shifts focus to the built-in security features of Windows: Windows Update, Windows Security (antivirus, firewall, app control, device security), BitLocker, and the Volume Shadow Copy Service.

Understanding these tools matters in cybersecurity because they are the first layer of defense on any Windows endpoint, and attackers routinely target, disable, or evade them (for example, disabling real-time protection or deleting shadow copies before deploying ransomware).

## Task 1 — Introduction

### Explanation

I started the lab machine and connected to it via Remote Desktop using the administrator credentials provided, accepting the certificate warning to reach the remote system, the same setup process as Parts 1 and 2.

## Task 2 — Windows Updates

### Explanation

Windows Update is Microsoft's service for delivering security patches, feature updates, and fixes for Windows and other Microsoft products like Defender. Updates are typically released on **Patch Tuesday**, the second Tuesday of each month, but Microsoft can push urgent/critical patches outside that schedule if needed.

Since Windows 10, updates can be postponed but not indefinitely ignored — the system will eventually reboot and apply them, which was a deliberate change by Microsoft to keep devices patched, since many users historically delayed updates to avoid a reboot.

On the attached lab machine, the Windows Update settings were shown as "managed" (usually only seen in managed/corporate environments, not on typical home devices), and no updates were available since the VM has no internet access to reach Microsoft's servers.

### Commands / Tools

```text
control /name Microsoft.WindowsUpdate
```

**Explanation:** Opens the Windows Update settings page directly from the Run dialog or Command Prompt, without needing to navigate through the Settings app manually.

### Questions

**Question:** There were two definition updates installed in the attached VM. On what date were these updates installed?

**Answer:** *This depends on the specific date shown in the Windows Update history on your deployed VM — I don't have that value recorded. Please check Settings > Windows Update > Update history on your machine and let me know the date so I can add it here.*

## Task 3 — Windows Security

### Explanation

Windows Security is the central dashboard for managing Windows' built-in protection tools. It's organized into protection areas:

- Virus & threat protection
- Firewall & network protection
- App & browser control
- Device security

Each area shows a status icon:

- **Green** — sufficiently protected, no action needed
- **Yellow** — a safety recommendation to review
- **Red** — something needs immediate attention

The lab machine, being Windows Server 2019, displays this dashboard slightly differently from a Windows 10 Home or Pro edition.

### Questions

**Question:** Checking the Security section on your VM, which area needs immediate attention?

**Answer:** *This is specific to the status shown on your deployed VM at the time you checked it — I don't have that recorded. Let me know which area showed a red/yellow icon and I'll fill this in.*

## Task 4 — Virus & Threat Protection

### Explanation

This section is split into two parts:

**Current threats**, which includes:

- **Quick scan** — checks the folders where threats are most commonly found
- **Full scan** — checks every file and running program on the hard disk (can take over an hour)
- **Custom scan** — lets me choose specific files/locations to check
- **Threat history** — shows the last scan, quarantined threats (isolated and prevented from running), and allowed threats (items flagged as threats that I've manually permitted to run — only safe to do if I'm certain about what the item is)

**Virus & threat protection settings**, which includes:

- **Real-time protection** — actively detects and blocks malware from installing or running
- **Cloud-delivered protection** — uses up-to-date threat data from Microsoft's cloud for faster detection
- **Automatic sample submission** — sends suspicious sample files to Microsoft to help improve threat detection for everyone
- **Controlled folder access** — blocks unauthorized/unknown applications from modifying protected folders; only trusted, approved apps can make changes when this is on
- **Exclusions** — lets specific files or folders be skipped during scans, useful for reducing false positives, but risky if misused since excluded items are never scanned
- **Notifications** — alerts about the device's security health

**Ransomware protection** depends on Controlled folder access being enabled, which in turn requires Real-time protection to be enabled.

On the lab machine, Real-time protection is intentionally turned off, since the VM has no internet access and there are no real threats present, so this is safe in this specific lab context. On a personal device, real-time protection should always stay enabled unless a third-party antivirus is providing equivalent protection.

### Questions

**Question:** Specifically, what is turned off that Windows is notifying you to turn on?

**Answer:** Real-time protection.

## Task 5 — Firewall & Network Protection

### Explanation

A firewall controls what traffic is and isn't allowed to pass through a device's network ports — essentially acting as a checkpoint for anything trying to enter or leave the system.

Windows Firewall offers three separate profiles, each with its own settings:

| Profile | Used for |
|---|---|
| **Domain** | Networks where the machine can authenticate to a domain controller |
| **Private** | User-assigned profile for private/home networks |
| **Public** | Default profile for public networks like coffee shop or airport Wi-Fi |

Each profile can independently be turned on/off, and each can be set to block all incoming connections. It's also possible to view and manage which apps are allowed through the firewall for each profile.

### Commands / Tools

```text
WF.msc
```

**Explanation:** Opens Windows Defender Firewall with Advanced Security directly, which is the deeper configuration interface for firewall rules (aimed at advanced users).

### Questions

**Question:** If you were connected to airport Wi-Fi, what most likely will be the active firewall profile?

**Answer:** Public — since airport Wi-Fi is an open/public network, Windows defaults to the Public profile, which applies the most restrictive settings by default.

## Task 6 — App & Browser Control

### Explanation

This section manages Microsoft Defender SmartScreen, which protects against phishing and malware websites, malicious applications, and potentially harmful file downloads. SmartScreen's status can be set to **Warn**, **Block**, or **Off**.

Two features covered here:

- **Check apps and files** — SmartScreen checks unrecognized apps and files downloaded from the web before they run
- **Exploit protection** — built into Windows (Windows 10 and Windows Server 2019 both include it) to help defend against exploitation techniques attackers use against running processes

Microsoft's own recommendation is to leave these settings at their defaults unless I'm certain about what I'm changing.

## Task 7 — Device Security

### Explanation

This section covers hardware-based security features:

- **Core isolation (Memory integrity)** — prevents malicious code from being inserted into high-security processes by isolating them in a protected area of memory
- **Security processor** — relates to the **Trusted Platform Module (TPM)**, a hardware-based security chip

### Questions

**Question:** What is the TPM?

**Answer:** The Trusted Platform Module is a dedicated hardware chip designed to perform cryptographic operations and provide hardware-based security functions. It has physical tamper-resistant mechanisms built in, meaning malicious software cannot interfere with its security functions.

## Task 8 — BitLocker

### Explanation

BitLocker Drive Encryption is a Windows data protection feature that encrypts entire drives to defend against data theft or exposure if a device is lost, stolen, or improperly decommissioned. It provides the strongest protection when paired with a TPM (version 1.2 or later), since the TPM works with BitLocker to protect data and confirm the system hasn't been tampered with while offline.

BitLocker was not available on the attached lab VM, since it's running Windows Server 2019 without this feature enabled in the lab environment.

### Questions

**Question:** We should use a removable drive on systems without a TPM version 1.2 or later. What does this removable drive contain?

**Answer:** A startup key, which the system needs in order to unlock and access the BitLocker-encrypted drive on machines that don't have a compatible TPM.

## Task 9 — Volume Shadow Copy Service (VSS)

### Explanation

The Volume Shadow Copy Service creates a **shadow copy** (also called a snapshot or point-in-time copy) of data for backup purposes. These snapshots are stored in the System Volume Information folder on each drive with protection enabled.

When VSS/System Protection is turned on, this allows for:

- Creating a restore point
- Performing a system restore
- Configuring restore settings
- Deleting restore points

From a security standpoint, this is genuinely important: ransomware authors are aware of VSS and often write their malware specifically to find and delete shadow copies, removing the possibility of recovery unless an offline or off-site backup exists separately.

### Questions

**Question:** What is VSS?

**Answer:** The Volume Shadow Copy Service — a Windows service that coordinates the creation of consistent, point-in-time snapshots (shadow copies) of data, primarily used for backup and system restore purposes.

## Key Takeaways

- Learned how Windows Update patches the system on a predictable schedule (Patch Tuesday) but can push urgent fixes outside that cycle.
- Understood the layout and purpose of the Windows Security dashboard and its four protection areas.
- Learned the difference between Quick, Full, and Custom antivirus scans, and how Controlled Folder Access ties into ransomware protection.
- Understood the three Windows Firewall profiles (Domain, Private, Public) and when each applies.
- Learned what SmartScreen and Exploit Protection do to guard against malicious files and exploitation attempts.
- Understood what a TPM is and how it strengthens BitLocker encryption.
- Learned why VSS/shadow copies matter for backup and recovery, and why ransomware specifically targets them.

## Conclusion

This room gave me a working understanding of the security tools built directly into Windows, from patching and antivirus through to firewall profiles, encryption, and backup snapshots. The most important concept I'm taking away is how these features interact with real attacker behavior — disabling real-time protection, evading SmartScreen, or deleting shadow copies are all things attackers specifically do, so knowing how these protections work (and how they can be turned off) is directly useful for both defensive and offensive security work going forward.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
