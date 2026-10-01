# SIEM

I completed the **SIEM** room on TryHackMe, where I learned how Security Information and Event Management systems are used by security analysts to collect, organize, correlate, and investigate security logs.

Before this room, I understood logs mainly as records of activities happening on individual systems. This room helped me understand the bigger picture. In a real network, there can be many different devices generating logs at the same time, and manually checking each device would make investigation slow and inefficient. A SIEM provides a centralized place where these logs can be collected and analyzed.

**SIEM** stands for **Security Information and Event Management**.

## Understanding Log Sources

I learned that devices in a network continuously generate logs whenever activities occur. These devices are referred to as **log sources**.

I learned to distinguish between two major categories of log sources: **host-centric** and **network-centric**.

Host-centric log sources generate information about activities occurring on a particular host. Examples include Windows machines, Linux machines, and servers. The events can include a user attempting to authenticate, accessing a file, executing a process, or modifying a registry key.

Network-centric log sources generate information about communication between systems or between a system and the Internet. Examples include firewalls and routers. Their logs can contain information about network connections, web traffic, file-sharing activity, and access to network resources.

This helped me understand that a security analyst does not investigate only what happens inside a computer. Network communication also provides important evidence about what is happening in an environment.

## Why Isolated Logs Are Difficult to Investigate

One of the most important things I learned was why having logs available does not automatically make investigation easy.

A network can have a large number of log sources, and each source can generate hundreds of events per second. If I had to investigate an incident by connecting to every individual machine and checking its logs separately, the process would become extremely time-consuming.

I also learned about the problem of **limited context**.

An individual event might look completely normal when viewed by itself. For example, a user accessing a file is not necessarily suspicious. However, when I correlate that event with other events, the situation can become much more interesting.

For example, if I see that a user first gained access to another machine through lateral movement, then accessed a sensitive file, the file access event has a different meaning when viewed together with the other events.

This taught me that security investigation is not always about finding one obviously malicious event. Sometimes the evidence comes from connecting several individually normal events together.

I also learned that manually analyzing all events is unrealistic because of the sheer volume of data. An analyst could easily miss important activity while going through thousands or millions of events.

Another problem is **different log formats**. Windows, Linux, firewalls, web servers, and other devices do not necessarily produce logs in the same format. This makes analysis more difficult when the analyst has to understand every format separately.

## What a SIEM Does

I learned that a SIEM addresses these problems by providing a centralized security monitoring and analysis platform.

The basic idea I took from the room is that a SIEM collects logs from different sources, brings them into a centralized location, parses and normalizes them, correlates related events, and uses detection rules to identify potentially malicious activity.

One of the first capabilities I learned about was **centralized log collection**.

Instead of logging into every endpoint, server, firewall, or other device individually, logs can be forwarded to the SIEM. This gives the analyst a central location from which to monitor activity across the environment.

I then learned about **log parsing and normalization**.

Parsing involves breaking a raw log into individual fields so that the information can be understood and searched more easily. Normalization involves converting information from different log sources into a consistent format.

This is important because a Windows log and a Linux log can look very different. A SIEM needs to make the information usable in a consistent way so that detection rules and searches can operate effectively.

## Correlation

The concept of **log correlation** was particularly important to me.

I learned that a SIEM can examine events from different sources and identify relationships between them.

For example, I looked at a scenario where a user logs in from an IP address they have not previously used, accesses documents on a shared drive, executes a script, and then establishes an outbound network connection.

Each event by itself might not immediately prove malicious activity. When they are correlated, however, the sequence can indicate a possible compromise and data exfiltration.

This showed me why SIEM is more than simply storing logs. The value comes from being able to connect events and identify patterns that would be difficult to notice when looking at individual log sources.

## Alerting

I learned that SIEM platforms use **detection rules** to identify activity that meets specific conditions.

A detection rule is essentially a logical condition that tells the SIEM when it should generate an alert.

For example, a rule could be configured so that if a user has five failed login attempts within ten seconds, an alert is generated for multiple failed login attempts.

Another rule could detect a successful login after multiple failed attempts.

I also learned that detection rules can be created around specific Windows events. For example, I learned that **Windows Event ID 104** can be associated with clearing event logs, which can be used as part of a detection rule.

I also learned about **Windows Event ID 4688**, which is associated with process creation. A rule can examine this event and look at fields such as `NewProcessName` to detect specific process execution.

The example I studied was a rule that could identify the execution of the `whoami` command.

The important lesson I took from this is that detection rules depend heavily on the fields and values contained in normalized logs. If the SIEM cannot properly identify the relevant fields, creating reliable detection rules becomes much more difficult.

## Log Sources and Ingestion

I also learned how different systems store their logs before those logs reach the SIEM.

On **Windows**, I learned about the **Event Viewer**, where Windows events can be viewed. Windows assigns event IDs to different types of activities, which helps analysts identify and investigate particular events.

On **Linux**, I learned that logs are commonly stored under `/var/log`. Some examples I encountered included:

- `/var/log/httpd` for HTTP-related logs
- `/var/log/cron` for cron-related events
- `/var/log/auth.log` and `/var/log/secure` for authentication-related logs
- `/var/log/kern` for kernel-related events

I also learned that **web servers** generate logs containing information about requests, responses, errors, and other web activity. Apache logs, for example, can commonly be found under locations such as `/var/log/apache` or `/var/log/httpd`.

This helped me connect what I had previously learned about Linux and network activity with SIEM monitoring.

## How Logs Reach the SIEM

I learned several ways that logs can be ingested into a SIEM.

One method is through an **agent or forwarder**. A lightweight program can be installed on an endpoint and configured to collect important logs and forward them to the SIEM.

Another method is **Syslog**. I learned that Syslog is a widely used protocol for collecting and forwarding log information from systems such as servers and network devices to a centralized destination.

I also learned about **manual log upload**, where offline log data can be uploaded into some SIEM platforms for analysis.

Another method is **port forwarding**, where the SIEM listens on a particular port and endpoints forward their log data to that listening port.

Understanding these ingestion methods helped me see that collecting logs is an actual process that has to be configured. The SIEM does not simply receive every log automatically without any setup.

## Alert Investigation

I learned that generating an alert is only the beginning of the investigation.

When an alert is triggered, the analyst needs to examine the events associated with it and determine why the detection rule was triggered.

The analyst checks which conditions of the rule were satisfied and investigates the surrounding activity to determine whether the alert represents a genuine security incident.

I learned the difference between a **false positive** and a **true positive**.

A false positive occurs when an alert is generated even though the activity is legitimate. In that situation, the detection rule may need to be tuned to reduce similar unnecessary alerts in the future.

A true positive occurs when the alert corresponds to genuinely suspicious or malicious activity. In that case, the analyst continues with further investigation and response.

Depending on what is discovered, the response can include contacting the asset owner, isolating an affected host, or blocking a suspicious IP address.

## What I Took From the Practical Lab

The practical part of the room helped me connect the concepts to an actual SIEM interface.

I worked with the dashboard and events to investigate a suspicious activity scenario. I had to identify the process that caused the alert, determine the user responsible for the process execution, identify the hostname associated with the suspicious user, and examine the detection rule to determine which term matched the suspicious process.

I then had to determine whether the event represented a true positive or a false positive and select the appropriate action to obtain the flag.

This was useful because it changed the learning experience from simply understanding what a SIEM is to actually following the basic workflow of a security analyst.

I had to look at the alert, trace it back to the relevant event, examine the user and host involved, understand why the detection rule triggered, and then make a determination based on the available evidence.

## What I Understand Now

After completing this room, I understand that SIEM is fundamentally about giving security analysts **centralized visibility across many different sources of security data**.

I learned that logs are generated everywhere in a network. Endpoints, servers, network devices, firewalls, and web servers can all provide useful evidence. The problem is that these logs can be numerous, distributed across different systems, and stored in different formats.

A SIEM helps solve this by collecting the logs centrally, parsing and normalizing them, correlating related events, and applying detection rules to identify activity that requires investigation.

I also learned that an alert does not automatically mean that an attack has occurred. The analyst still has to investigate the evidence behind the alert and determine whether it is a true positive or false positive.

The biggest concept I took from the room is that **individual events can be difficult to interpret in isolation, while correlated events can reveal a much clearer picture of what is happening in a network**.

This room also gave me a practical foundation for understanding how SIEM fits into **SOC monitoring, detection, alert investigation, and incident response**.

## Room

**Platform:** TryHackMe  
**Room:** SIEM  
**Focus:** Security Information and Event Management, log sources, log ingestion, correlation, detection rules, alert investigation, and SIEM dashboards
