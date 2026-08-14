# Active Directory Basics

**Write-up by: Nicholas Kiyimba**

## Introduction

This room introduces Microsoft's Active Directory (AD), which is the backbone of identity and device management in most corporate Windows networks. It covers what a Windows domain is, the core object types stored in AD (users, machines, security groups), how to organise those objects using Organizational Units (OUs), how Group Policy Objects (GPOs) push configuration and security settings across a network, the two authentication protocols used in a domain (Kerberos and NetNTLM), and how multiple domains can be linked together through trees, forests, and trust relationships.

This is directly relevant to cybersecurity because Active Directory is one of the most heavily targeted components in a corporate network. A huge proportion of real-world attacks against organisations eventually involve compromising or abusing AD in some way (privilege escalation, lateral movement, credential theft), so understanding how it's structured and how it's meant to work is a prerequisite to understanding how it gets attacked.

## Task 1 — Introduction

### Explanation

This task was a short overview of the room's objectives: understanding what Active Directory and an AD domain are, what components make up a domain, and how forests and domain trust work. No lab work was required here.

## Task 2 — Windows Domains

### Explanation

The core problem this task explains is scale. Managing a handful of computers individually (configuring each one by hand, walking over to fix issues on-site) is manageable for a tiny business, but becomes completely impractical once a company grows to hundreds of computers and users across multiple offices.

A **Windows domain** solves this by centralising administration: it's a group of users and computers managed together under a single repository called **Active Directory**. The server running the AD services is called a **Domain Controller (DC)**.

The two main benefits of a domain setup are:

- **Centralised identity management** – user accounts are configured once in AD rather than separately on every machine
- **Centralised security policy management** – policies can be configured in AD and applied across every relevant user or computer on the network

A real-world example given was school or university networks: the same username and password work on any campus computer because each machine forwards the authentication request back to Active Directory to verify the credentials, rather than storing accounts locally on every machine. This is also how a university can prevent students from accessing the Control Panel on lab machines — that restriction is a policy pushed out from AD.

For the rest of the room, I took on the role of a new IT admin at a company called THM Inc., with administrative access to their pre-configured Domain Controller for the THM.local domain.

### Questions

**Question:** In a Windows domain, credentials are stored in a centralised repository called...

**Answer:** Active Directory (AD).

**Question:** The server in charge of running the Active Directory services is called...

**Answer:** A Domain Controller (DC).

## Task 3 — Active Directory (Objects)

### Explanation

Active Directory works as a catalogue of "objects" that exist on the network. The main object types covered were:

**Users** – one of the most common object types, and one of a category called **security principals**, meaning they can be authenticated by the domain and assigned privileges over resources like files or printers. Users can represent:

- **People** – actual staff who need network access
- **Services** – accounts used to run services (e.g. IIS or MSSQL), which only get the specific privileges needed to run that service, rather than broad access

**Machines** – every computer that joins the domain gets its own machine object, which is also a security principal with its own account. Machine accounts are local administrators on their own assigned computer, and are meant to only be used by that computer itself. Machine account names follow a predictable pattern: the computer's name followed by a `$`, so a computer named `TOM-PC` would have a machine account called `TOM-PC$`. Their passwords are automatically rotated and are typically 120 random characters long, making them far stronger than a typical user password.

**Security Groups** – also security principals, these let permissions be assigned to a whole group at once rather than to individual users one by one. Groups can contain users, machines, and even other groups. Some of the most important default groups are:

| Security Group | Description |
|---|---|
| Domain Admins | Administrative privileges over the entire domain, including every computer and the DCs |
| Server Operators | Can administer Domain Controllers, but can't change administrative group memberships |
| Backup Operators | Can access any file regardless of permissions, used to perform backups |
| Account Operators | Can create or modify other accounts in the domain |
| Domain Users | Includes every user account in the domain |
| Domain Computers | Includes every computer in the domain |
| Domain Controllers | Includes every DC in the domain |

**Active Directory Users and Computers (ADUC)** is the main tool for managing these objects, opened from the Start Menu on the Domain Controller. It shows the hierarchy of users, computers, and groups, organised into **Organizational Units (OUs)** — container objects used to group users/machines that should share the same policies. A user can only be a member of one OU at a time. On the lab DC, there was already a `THM` OU containing sub-OUs for IT, Management, Marketing, R&D, and Sales, mirroring the business's actual department structure, which is a common and sensible way to organise OUs.

Aside from custom OUs, Windows creates some default containers automatically:

- **Builtin** – default groups available on any Windows host
- **Computers** – where any machine joining the domain lands by default
- **Domain Controllers** – default OU containing the DCs
- **Users** – default domain-wide users and groups
- **Managed Service Accounts** – accounts used by services in the domain

### OUs vs Security Groups

Even though both OUs and groups are used to "organise" objects, their purposes are different:

- **OUs** are for applying policies — since a user can only be in one OU, it makes sense that policies applied by OU membership represent a single, consistent set of rules for that user
- **Security Groups** are for granting permissions over resources (like a shared folder) — a user can belong to many groups at once, which is exactly what's needed since someone might need access to multiple different resources

### Questions

**Question:** Which group normally administrates all computers and resources in a domain?

**Answer:** Domain Admins.

**Question:** What would be the name of the machine account associated with a machine named TOM-PC?

**Answer:** `TOM-PC$`

**Question:** Suppose our company creates a new department for Quality Assurance. What type of containers should we use to group all Quality Assurance users so that policies can be applied consistently to them?

**Answer:** An Organizational Unit (OU).

## Task 4 — Managing Users and the AD

### Explanation

This task was hands-on practice inside Active Directory Users and Computers, working from an organisational chart provided in the room to bring the domain's actual OU/user structure in line with what the business chart said it should be.

An extra department OU existed in the domain that had been closed due to budget cuts and needed to be removed. By default, OUs in AD are protected against accidental deletion, so trying to delete one directly gives an error. To actually delete it, I needed to:

1. Enable **Advanced Features** from the View menu in ADUC (this reveals additional containers and options)
2. Right-click the OU → Properties → Object tab → uncheck the box that protects the object from accidental deletion
3. Delete the OU as normal, confirming that all users, groups, and sub-OUs under it would also be removed

After removing the extra department, I compared the remaining OUs against the organisational chart and created/deleted users as needed so the AD structure matched the chart exactly.

### Delegation

**Delegation** is the process of granting a specific user privileges to perform certain administrative tasks over an OU, without needing to make them a full Domain Admin. A common real-world use case is giving IT support staff the ability to reset passwords for regular users in specific departments.

Following the room's example, I delegated control of the Sales OU to a user named Phillip (right-click the OU → Delegate Control → select Phillip as the user → choose the "reset user passwords" task). Once delegated, I tested this by logging in as Phillip via RDP (`THM\phillip`) and using PowerShell (since Phillip doesn't have permission to open ADUC itself) to reset another user's (Sophie's) password:

```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
```

**Explanation:** Resets the target user's (`sophie`) AD account password. `-Reset` indicates this is an administrative reset rather than the user changing their own password. `-NewPassword` takes a secure string, prompted for interactively here via `Read-Host -AsSecureString`, so the plaintext password is never typed directly into the command or shown on screen. `-Verbose` prints details of the operation being performed, useful for confirming exactly which AD object was affected.

```powershell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

**Explanation:** Forces the specified user (`sophie`) to change their password the next time they log in. This is good practice after an administrative password reset, since it means the temporary password set by the administrator (Phillip, in this case) isn't left in permanent use.

After resetting Sophie's password, I logged into her account via RDP (`THM\sophie`) using the new password and retrieved a flag from her desktop, confirming the delegated reset had worked end-to-end.

### Questions

**Question:** What was the flag found on Sophie's desktop?

**Answer:** *This is a specific value generated on your lab instance — I don't have it recorded. Let me know the flag text from Sophie's desktop and I'll add it here.*

**Question:** The process of granting privileges to a user over some OU or other AD Object is called...

**Answer:** Delegation.

## Task 5 — Managing Computers in Active Directory

### Explanation

By default, every machine that joins the domain (aside from Domain Controllers) lands in the default **Computers** container. Leaving everything there isn't ideal, since servers and regular user workstations usually need very different policies applied to them.

A sensible starting point is to split devices into at least three categories:

1. **Workstations** – the machines regular users actually log into and work from day-to-day; these should never have a privileged account signed into them
2. **Servers** – machines that provide services to users or to other servers
3. **Domain Controllers** – manage the AD domain itself, and are considered the most sensitive machines in the network since they store hashed passwords for every user account

Since Domain Controllers already get their own OU automatically, I created two new OUs directly under the `thm.local` domain: `Workstations` and `Servers`. I then moved the existing personal computers and laptops out of the default Computers container into the Workstations OU, and moved the servers into the Servers OU. This sets things up so that different policies can later be applied specifically to workstations versus servers.

### Questions

**Question:** After organising the available computers, how many ended up in the Workstations OU?

**Answer:** *This depends on the specific machines present in your lab instance's Computers container — I don't have that count recorded. Let me know the number and I'll add it here.*

**Question:** Is it recommendable to create separate OUs for Servers and Workstations? (yay/nay)

**Answer:** Yay — separating them allows different, appropriate policies (e.g. security baselines, lockout screens, access restrictions) to be applied to each category independently, rather than forcing one identical policy set onto every machine in the domain.

## Task 6 — Group Policies

### Explanation

The whole point of organising users and computers into OUs is so that different policies can be applied to each group. In Windows, this is done through **Group Policy Objects (GPOs)** — collections of settings that get applied to an OU. A GPO can contain **Computer Configuration** settings, **User Configuration** settings, or both, letting administrators set baselines for either machines or identities (or both at once).

GPOs are configured using the **Group Policy Management** tool from the Start Menu. Opening it shows the full OU hierarchy. To apply a policy, a GPO first needs to be created under Group Policy Objects, then **linked** to the OU(s) where it should apply. On the lab DC, three GPOs already existed:

- **Default Domain Policy** – linked to the `thm.local` domain as a whole
- **RDP Policy** – also linked to the domain as a whole
- **Default Domain Controllers Policy** – linked only to the Domain Controllers OU

An important behaviour to understand: a GPO applies to the OU it's linked to **and** to any sub-OUs beneath it. So a GPO linked at the domain root (like Default Domain Policy) still affects the Sales OU, even though Sales wasn't linked directly.

Each GPO has a **Scope** tab (showing where it's linked, and any Security Filtering restricting which users/computers it applies to — by default this is the `Authenticated Users` group, meaning everyone) and a **Settings** tab (the actual configuration it enforces). The Default Domain Policy, for example, only contained Computer Configuration settings, covering basic password and account lockout policy.

I edited the Default Domain Policy to change the minimum password length requirement to 10 characters, navigating to:

```text
Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy
```

Since this GPO is linked at the domain level, this change would apply to every computer in the domain.

### GPO Distribution

GPOs are distributed to machines on the network via a share called **SYSVOL**, hosted on the Domain Controller. This share maps by default to `C:\Windows\SYSVOL\sysvol\` on the DC, and all domain-joined computers periodically sync with it to pick up policy changes. A change to a GPO can take up to 2 hours to propagate naturally. To force an individual machine to sync immediately, the following command is run on that machine:

```powershell
gpupdate /force
```

**Explanation:** Forces an immediate refresh of both Computer and User Group Policy settings on the local machine, rather than waiting for the normal background refresh interval.

### Creating GPOs for THM Inc.

Two specific GPOs were required for this task:

**1. Restrict Control Panel Access** — a new GPO restricting Control Panel access to IT department users only. Since this needed to target specific users rather than machines, the relevant setting was under **User Configuration**, enabling the **Prohibit Access to Control Panel and PC settings** policy. Once configured, the GPO was linked to the Marketing, Management, and Sales OUs (but deliberately not IT), so only those departments would be restricted.

**2. Auto Lock Screen** — a new GPO to automatically lock workstations and servers after 5 minutes of inactivity. Rather than linking this separately to the Workstations, Servers, and Domain Controllers OUs, it was instead linked once at the **root domain** level, since all of those OUs are children of the root and therefore inherit its policies. Since this GPO's setting (machine inactivity limit) is a Computer Configuration, it has no effect on OUs like Sales or Marketing that only contain users, even though those OUs technically inherit the GPO too — the relevant user-based settings simply don't apply since there are no matching Computer Configuration targets in a user-only OU. The specific setting is located under the machine inactivity limit policy, set to 5 minutes.

To verify both GPOs worked, I logged in via RDP as Mark (a Marketing department user) and confirmed that opening the Control Panel returned a message that the action was denied by the administrator. Since the Control Panel GPO wasn't linked to IT, IT department users remain unaffected and can still access it. If a GPO doesn't appear to be working immediately after being linked, running `gpupdate /force` on the target machine resolves this.

### Questions

**Question:** What is the name of the network share used to distribute GPOs to domain machines?

**Answer:** SYSVOL.

**Question:** Can a GPO be used to apply settings to users and computers? (yay/nay)

**Answer:** Yay — a single GPO can contain both User Configuration and Computer Configuration settings.

## Task 7 — Authentication Methods

### Explanation

In a Windows domain, all credentials are stored on the Domain Controller. Whenever a user authenticates to any service using domain credentials, that service has to check with the DC to confirm the credentials are valid. Two protocols handle this:

- **Kerberos** – the default authentication protocol on any modern Windows domain
- **NetNTLM** – a legacy protocol kept around mainly for backward compatibility; considered obsolete but still commonly enabled alongside Kerberos on most networks

**Kerberos Authentication** works using tickets, which act as proof that a user has already authenticated successfully:

1. The user sends their username and a timestamp, encrypted with a key derived from their password, to the **Key Distribution Center (KDC)** — a service that normally runs on the Domain Controller. The KDC responds with a **Ticket Granting Ticket (TGT)**, which lets the user request further tickets without needing to resend their credentials each time, along with a Session Key needed for future requests. The TGT itself is encrypted using the `krbtgt` account's password hash, so the user can't actually read its contents — they can only present it.
2. When the user wants to connect to an actual service (a share, website, database, etc.), they present their TGT to the KDC to request a **Ticket Granting Service (TGS)** ticket, specific to that one service. This request includes their username, a timestamp encrypted with the Session Key, the TGT itself, and a **Service Principal Name (SPN)** identifying exactly which service/server they want to reach. The KDC responds with a TGS and a Service Session Key. The TGS is encrypted using a key derived from the hash of the account the target service runs under (the Service Owner).
3. The user presents the TGS directly to the desired service. The service decrypts the TGS using its own account's password hash to validate the Service Session Key and complete the authentication.

**NetNTLM Authentication** uses a simpler challenge-response mechanism instead of tickets:

1. The client sends an authentication request to the server
2. The server generates a random challenge and sends it to the client
3. The client combines its password hash with the challenge to compute a response, and sends that response back
4. The server forwards the challenge and response to the Domain Controller for verification
5. The DC recalculates the expected response using the challenge and compares it to what the client sent — if they match, authentication succeeds
6. The result is passed back through the server to the client

Importantly, the user's actual password (or hash) is never transmitted over the network at any point in this process — only the challenge and the computed response are sent. Note this challenge-response flow only requires the DC's involvement when using a domain account; for a local account, the server itself can verify the response since it already stores the password hash locally in its SAM database.

### Why this matters for cybersecurity

Since Kerberos is the default and NetNTLM is legacy-but-usually-still-enabled, NetNTLM tends to be a common target for attackers, since its challenge-response mechanism is vulnerable to relay and cracking attacks in ways that Kerberos generally isn't. Knowing which protocol is in use, and that most networks leave both enabled, is a foundational piece of knowledge for understanding common AD attack paths later on.

### Questions

**Question:** Will a current version of Windows use NetNTLM as the preferred authentication protocol by default? (yay/nay)

**Answer:** Nay — Kerberos is the default and preferred protocol; NetNTLM is only kept for legacy compatibility.

**Question:** When referring to Kerberos, what type of ticket allows us to request further tickets known as TGS?

**Answer:** The Ticket Granting Ticket (TGT).

**Question:** When using NetNTLM, is a user's password transmitted over the network at any point? (yay/nay)

**Answer:** Nay — only the challenge and the computed response are sent; the password/hash itself never travels over the network.

## Task 8 — Trees, Forests, and Trusts

### Explanation

As a company grows, a single domain can eventually become difficult to manage, especially if different branches (e.g. different countries with different regulations) need to be administered somewhat independently.

**Trees** – if two or more domains share the same root namespace (e.g. `thm.local`), they can be joined into a tree. For example, `thm.local` could have subdomains `uk.thm.local` and `us.thm.local`, each with its own AD, computers, and users. This gives each branch's IT team full control over their own DC and users, without being able to manage the other branch's resources — while still being part of the same overall tree. A new group relevant here is **Enterprise Admins**, which grants administrative privileges across every domain in the enterprise (as opposed to Domain Admins, whose privileges are limited to a single domain).

**Forests** – if a company merges with or acquires another company that has a completely different domain namespace, the union of multiple trees with different namespaces is called a forest.

**Trust Relationships** – domains organised into trees and forests still need a way for users in one domain to access resources in another (e.g. a user in `THM UK` needing to access a file share hosted in `MHT ASIA`). This is done through trust relationships:

- A **one-way trust** — if Domain AAA trusts Domain BBB, then users from BBB can be authorised to access resources in AAA. Notably, the direction of trust is the opposite of the direction of access.
- A **two-way trust** — both domains mutually trust each other, letting users from either domain be authorised in the other. Joining domains into a tree or forest creates two-way trusts by default.

Importantly, a trust relationship on its own doesn't automatically grant access to everything in the other domain — it just makes it possible to authorise specific users for specific resources across the trust; the actual authorisation still has to be configured deliberately.

### Questions

**Question:** What is a group of Windows domains that share the same namespace called?

**Answer:** A tree.

**Question:** What should be configured between two domains for a user in Domain A to access a resource in Domain B?

**Answer:** A trust relationship.

## Task 9 — Conclusion

### Explanation

This room was an introduction to the core concepts of Active Directory and Windows domains, not a full production deployment guide. For going deeper, the room pointed to the Active Directory Hardening room (for securing an AD environment) and the Compromising Active Directory module (for how attackers actually abuse AD misconfigurations).

## Key Takeaways

- Learned why Windows domains exist: centralising identity and policy management at scale, instead of configuring every machine individually.
- Understood the core AD object types — users, machine accounts, and security groups — and that all three are "security principals."
- Learned the difference between OUs (used for applying policies, one per user) and Security Groups (used for granting resource permissions, many per user).
- Practiced real AD administration: deleting a protected OU, syncing users to an org chart, and delegating specific admin rights (password resets) to a non-admin user.
- Learned how to organise computer objects into Workstations, Servers, and Domain Controllers OUs for more precise policy targeting.
- Understood how GPOs are created, linked, and inherited down the OU tree, and built two working GPOs (Control Panel restriction and auto-lock screen).
- Learned the mechanics of both Kerberos (ticket-based) and NetNTLM (challenge-response) authentication, and why Kerberos is preferred from a security standpoint.
- Understood how trees, forests, and trust relationships let multiple domains coexist and share resources in a controlled way.

## Conclusion

This room gave me a solid foundational understanding of how Active Directory actually works under the hood, not just conceptually. Working hands-on with OUs, delegation, GPOs, and password resets made the theory concrete, and understanding Kerberos versus NetNTLM in particular feels like an important building block, since so much of real-world AD attack tooling revolves around abusing exactly these authentication mechanisms. This has given me a much clearer picture of what I'll need to understand before moving into more offensive AD-focused rooms.

---
*This write-up reflects my own understanding and notes from completing this room on TryHackMe.*
