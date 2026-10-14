# TryHackMe Incident Response Fundamentals

## Introduction to Incident Response

Incident response is the organised process of responding when a security incident occurs. Preventive controls such as security guards, CCTV cameras, locks, and other protective measures help reduce the chance of an incident, but they do not remove the need to respond when an incident succeeds. In cybersecurity, incident response defines the actions an organisation takes after a potentially harmful event has been identified.

Incident response teams are often part of a Security Operations Centre (SOC). Their role is to investigate alerts, determine whether malicious activity has occurred, and coordinate an appropriate response. Preparation is important because a team needs plans, people, processes, and security tools ready before an incident happens.

## Events, Alerts, and Incidents

Computers and other devices perform activities initiated by users and activities that run in the background. Examples include opening emails, browsing websites, downloading files, and running system processes. Many of these activities can generate event records, which are commonly stored in logs. Logs provide information that investigators can review to understand what happened on a system.

The volume of log data can be too large for analysts to inspect manually. A Security Information and Event Management (SIEM) solution collects and correlates log data from multiple sources and helps identify activity that may require investigation. It filters and prioritises relevant information so analysts can focus on suspicious events rather than reviewing every record individually.

An alert is generated when a security solution identifies activity that may be harmful. An alert can be a false positive or a true positive. A false positive occurs when an alert identifies activity as suspicious even though the activity is legitimate. For example, a large transfer of data might trigger an alert even though it is an authorised backup to cloud storage. A true positive occurs when an alert correctly identifies harmful activity, such as a phishing email containing a malicious attachment that a user downloads.

In the terminology used in this room, a true positive that confirms harmful activity is treated as an incident. Security teams can reduce noise by tuning detection rules and filtering known false positives. Confirmed incidents can then be prioritised according to severity, such as low, medium, high, or critical.

An alert is triggered after an event or group of events points to potentially harmful activity. When a security solution correctly identifies harmful activity, the alert is a true positive. If a fire alarm is triggered by smoke from cooking when there is no fire, the example represents a false positive.

## Types of Security Incidents

Describing an event simply as a hack is vague. Naming the incident type more precisely helps an organisation understand what has happened and choose an appropriate response.

A phishing incident involves deceptive messages intended to trick recipients into taking an unsafe action, such as opening a malicious attachment or following a harmful link. A malware infection occurs when malicious software reaches and runs on a system, often after a user opens or executes a downloaded file.

A security breach involves unauthorised access to systems or confidential information. A data leak involves confidential information being exposed to people who should not have access to it. A data leak can happen accidentally, such as when an employee sends sensitive information to the wrong recipients or uses reply-all unintentionally. The distinction presented in this room is that a security breach involves intentional unauthorised access, while a data leak can also result from an accidental disclosure.

An insider attack occurs when someone with internal access, such as an employee, misuses that access to harm an organisation. One example is a disgruntled employee deliberately infecting systems. A logic bomb is malicious code that remains inactive until specified conditions are met, after which it performs its intended action. An insider could plant a logic bomb that activates after a particular date or event.

A denial-of-service (DoS) attack attempts to make a system, network, or application unavailable to legitimate users. One way to do this is by overwhelming the target with a large volume of requests until its resources are exhausted or its service becomes unavailable.

When a user’s system is compromised after downloading an email attachment, the incident type in the room’s example is a malware infection. An attack intended to disrupt the availability of an application is a denial-of-service attack.

## The Incident Response Process

Incident response frameworks provide a structured sequence of activities for managing security incidents. The room introduces the SANS incident response framework and the NIST incident response framework. Their phases use somewhat different labels, but they cover similar activities.

## The SANS Incident Response Framework

### Preparation

Preparation involves establishing the plans, resources, tools, and procedures needed to respond to incidents. It can include deploying appropriate security solutions and providing security awareness training to employees. Employees who recognise suspicious activity and report it promptly can help the response team identify incidents sooner.

### Identification

Identification involves determining whether suspicious activity represents a security incident. Responders may examine network traffic, firewall logs, other system logs, and endpoint performance indicators. For example, a sudden and sustained increase in outbound data transfer or unexpected high CPU usage may indicate activity that needs investigation.

### Containment

Containment aims to limit the impact of an incident and prevent it from spreading. Responders might isolate an affected computer from the network, disable compromised accounts, or restrict access to part of a network. The appropriate action depends on the incident and the risk of further damage. Isolating a host from the network is an example of containment.

### Eradication

Eradication involves removing the threat from affected systems. Depending on the situation, responders might run a malware scan and remove malicious files, or wipe and rebuild a compromised system. The aim is to eliminate the cause of the incident rather than simply prevent it from spreading.

### Recovery

Recovery restores affected systems and services to normal operation. This may involve restoring a system from a known-good backup, reinstating user accounts, reconnecting a device to the network, and checking that the system is safe to use. Backups and recovery procedures can make this stage more reliable.

### Lessons Learned

Lessons learned takes place after the incident has been handled. The response team reviews what happened, how the incident was detected, whether the response was effective, and what could be improved. The findings can lead to changes in security controls, detection rules, training, and response procedures.

## The NIST Incident Response Framework

The NIST framework presented in the room groups the work into four broad phases. Preparation establishes readiness before an incident. Detection and analysis identifies suspicious activity and determines whether an incident has occurred. Containment, eradication, and recovery covers limiting the impact, removing the threat, and restoring affected systems. Post-incident activity reviews the response and identifies improvements.

The SANS lessons learned phase corresponds to NIST post-incident activity. When an organisation defines how it will apply a framework in its own environment, the resulting documented approach forms part of its incident response plan.

## Incident Response Techniques and Security Solutions

Manually identifying abnormal activity across large volumes of logs and live network traffic is difficult. Security solutions help collect, detect, and investigate suspicious behaviour.

A Security Information and Event Management (SIEM) solution centralises log data from multiple sources and analyses it to help identify possible incidents. Antivirus software detects or blocks known and suspected malware on a system. Endpoint Detection and Response (EDR) monitors endpoint activity and provides capabilities for detecting, investigating, and responding to threats.

Depending on its configuration and the evidence detected, an EDR solution may help contain a threat by isolating an endpoint from the network. Automated containment needs careful configuration because an incorrect detection could disrupt legitimate work. Security teams must balance rapid response against the operational impact of mistakenly isolating a healthy system.

## Playbooks and Runbooks

Organisations document response procedures so that responders can follow a consistent process. A playbook is a comprehensive guide for handling a particular incident type. It describes the sequence of activities from the beginning of the response through its completion.

A phishing-response playbook might instruct responders to notify relevant stakeholders, inspect the email header and body, check for attachments, determine which users opened the message or its attachment, isolate infected systems where necessary, and block the sender or otherwise prevent further delivery.

A runbook provides detailed execution instructions for a specific task within a wider process. A playbook might instruct a responder to analyse an email header, while the associated runbook explains the detailed steps for carrying out that analysis. A runbook generally covers a particular procedure rather than the entire incident lifecycle.

Comprehensive guidelines for responding to an incident are called playbooks. Detailed instructions for carrying out one specific action are characteristic of a runbook.

## Practical Lab: Responding to a Phishing Incident

The practical exercise simulates a phishing email that contains a malicious attachment and affects multiple hosts in a network. The task is to identify the threat vector, determine which devices downloaded the attachment, establish which devices executed the file, and respond to the affected systems.

The malicious email sender in the exercise is Jeff Johnson. The threat vector is the email attachment. Three devices downloaded the attachment, but only one device executed the file. The devices that downloaded the file without executing it could have the file quarantined to prevent later execution.

The host that executed the file required further investigation. The response workflow instructed the analyst to isolate that host from the network before reviewing its process timeline. The timeline showed activity involving `explorer.exe` and Chrome, followed by the download of a file named `place.pdf`. The file execution was followed by a malicious PowerShell process and a malicious DNS query.

After the investigation and response actions were completed, the exercise displayed the flag `THM{my_first_incident_response}`. This flag was the completion answer for the practical task.

## Conclusion

Incident response begins with identifying and validating suspicious activity, then applying an organised process to limit the impact, remove the threat, restore affected systems, and learn from the incident. The SANS framework describes six phases: preparation, identification, containment, eradication, recovery, and lessons learned. The NIST framework groups similar work into preparation, detection and analysis, containment, eradication and recovery, and post-incident activity.

SIEM, antivirus, and EDR solutions support detection and response, while playbooks and runbooks help teams carry out response procedures consistently. The practical exercise demonstrated how a phishing attachment can affect multiple hosts and how responders can distinguish between a downloaded file and an executed file, isolate a compromised host, and inspect the activity timeline.
