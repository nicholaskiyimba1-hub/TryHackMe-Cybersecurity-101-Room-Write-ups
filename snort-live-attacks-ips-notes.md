# Snort Live Attacks: Detecting and Blocking SSH and Reverse-Shell Traffic

## Introduction

I completed the TryHackMe **Snort Challenge - Live Attacks** room to practice examining network traffic and using Snort as an intrusion prevention system (IPS). The exercise presented two simulated incidents: repeated SSH connections consistent with a brute-force attack and suspicious outbound traffic consistent with a reverse shell. I used Snort 2.9.7.0 on the provided Linux lab machines. The main practical difference from passive monitoring was that I configured `drop` rules and observed Snort actively dropping matching packets.

This was a guided training environment, not an investigation of a real production network. The packet captures and command output documented below are the evidence for what I observed and practiced.

## Understanding Snort Detection and Prevention

Snort can inspect network packets and compare them against configured rules. In intrusion detection system (IDS) operation, matching traffic can produce alerts for an analyst to investigate. In intrusion prevention system (IPS) operation, Snort can also stop traffic that matches a `drop` rule, provided packets actually pass through a correctly configured inline inspection path.

I learned that a rule being loaded is not the same as a rule being triggered, and a rule being triggered is not always proof that traffic was blocked. During this exercise, the most useful confirmation of prevention was the `[Drop]` message in Snort's live output. I also used full alert logging so that detection events could be reviewed after execution.

The Snort version displayed by the lab was:

```text
Version 2.9.7.0 GRE (Build 149)
```

## Inspecting Live Network Traffic

I started by using Snort's packet sniffer mode to inspect live traffic on an interface:

```bash
sudo snort -v -i eth0
```

The `-v` option prints packet headers to the terminal, while `-i eth0` selects the network interface. I inspected the source and destination IP addresses, TCP source and destination ports, and the direction of repeated communication. I stopped the capture using `Ctrl + C` and reviewed the packet statistics.

In the first capture, Snort reported 5,621 packets, including 4,790 TCP packets and 72 UDP packets. It also reported 751 discarded packets and 2,376 bad checksums. These counters described the traffic and processing conditions, but they did not independently establish that an attack had occurred. In particular, checksum warnings can arise from capture or network offloading behavior and require context before being treated as malicious.

## Investigating Repeated SSH Connections

In the first scenario, I observed repeated connections from `10.10.245.36` to `10.10.140.29` on TCP port `22`. Some packets showed new TCP connection attempts, while others belonged to existing connections. The repeated connections matched the room's SSH brute-force scenario, although packet headers alone did not reveal individual password guesses or prove failed logins.

SSH, or Secure Shell, provides encrypted remote access and commonly uses TCP port 22. An attacker may attempt many authentication requests against an SSH service, but distinguishing brute-force activity from ordinary administration usually requires more evidence, such as authentication logs, timing, connection volume, and source behavior.

I edited the local Snort rule file:

```bash
sudo nano /etc/snort/rules/local.rules
```

The rule used for the first scenario was:

```snort
drop tcp 10.10.245.36 any -> 10.10.140.29 22 (msg:"SSH Brute Force Blocked"; sid:1000001; rev:1;)
```

The `drop` action instructs Snort to discard matching packets when it operates inline. The rule matches TCP packets from the identified source IP, using any source port, destined for TCP port 22 on the identified target. The `msg` field supplies an alert description, `sid` identifies the rule, and `rev` records its revision. This was a targeted lab rule, not a general-purpose brute-force detector: it matched the identified connection endpoints and port rather than counting failed authentication attempts.

I ultimately obtained the first scenario's flag after configuring inline prevention. The flag value is intentionally omitted because these notes are intended to document the technique rather than serve as a room answer sheet.

## Configuring Inline IPS with AFPacket

I initially attempted to start inline Snort using a single interface:

```bash
sudo snort -c /etc/snort/snort.conf -Q --daq afpacket -i eth0 -A console
```

Snort loaded one detection rule but failed to initialize AFPacket with this message:

```text
ERROR: Can't initialize DAQ afpacket (-1) - afpacket_daq_initialize: Invalid interface specification: 'eth0'!
```

I checked the available DAQ modules:

```bash
sudo snort --daq-list
```

The output included `afpacket(v5)` with live and inline capabilities. I then checked the network interfaces:

```bash
ip -br link
```

The first lab machine showed `eth0` and `eth1` as available interfaces. In this environment, the working command used the two-interface AFPacket specification:

```bash
sudo snort -c /etc/snort/snort.conf -Q --daq afpacket -i eth0:eth1 -A console
```

The `-c` option selects the Snort configuration file. The `-Q` option requests inline operation, `--daq afpacket` selects the AFPacket data acquisition module, and `-i eth0:eth1` supplies the interface pair used in this lab. The `-A console` option prints alerts to the terminal so I could verify whether the rule was matching traffic.

A later automated mission debrief recommended using `-i eth0` and described the paired syntax as a mistake. That recommendation conflicts with the actual error I received when using `eth0` alone and with my successful inline operation using `eth0:eth1`. AFPacket interface requirements depend on the Snort/DAQ version and deployment topology. I therefore record the command that worked in this particular lab rather than treating either interface form as universally correct. A real deployment also requires verifying that the selected interfaces are actually in the traffic path.

After testing, I ran Snort with full alert logging:

```bash
sudo snort -c /etc/snort/snort.conf -Q --daq afpacket -i eth0:eth1 -A full -l /var/log/snort
```

The `-A full` option uses Snort's full alert output format. The `-l /var/log/snort` option specifies the log directory. Full logging records alert information; it does not itself activate packet blocking. Blocking depends on inline operation, the configured rule action, and the traffic path. I learned to keep these separate functions in mind when troubleshooting.

## Investigating Suspicious Outbound Traffic

In the second scenario, I inspected traffic for signs of a reverse shell. A reverse shell is a connection initiated from a potentially compromised machine toward a remote listener, through which commands may be exchanged. Unlike an ordinary inbound remote-login connection, this arrangement can allow an attacker to interact with a host through an outbound connection.

The capture showed repeated TCP communication between `10.10.196.55` and `10.10.144.156`, with the latter using TCP port `4444`. Multiple source ports appeared on the connections from `10.10.196.55`. Port 4444 is commonly used in demonstrations involving Metasploit and reverse-shell listeners, but a port number by itself does not identify a specific tool or prove malicious activity. The room's scenario supplied the reverse-shell context.

I created a second targeted rule in the local rules file:

```snort
drop tcp 10.10.196.55 any -> 10.10.144.156 4444 (msg:"Reverse Shell Traffic Blocked"; sid:1000002; rev:1;)
```

This rule matches TCP packets leaving the suspected compromised host for the identified listener address on port 4444. It does not detect every possible reverse shell, because other connections can use different addresses, ports, or protocols.

I tested the rule using inline console output. Snort initialized successfully and produced repeated messages like this:

```text
[Drop] [**] [1:1000002:1] Reverse Shell Traffic Blocked [**] [Priority: 0] {TCP} 10.10.196.55:54124 -> 10.10.144.156:4444
```

This was important evidence: the output identified the rule by its SID and showed a `Drop` action for traffic matching the suspicious connection. It demonstrated that the configured rule was actively preventing those observed packets in the lab.

## Logging and Confirming Containment

After observing the drops in console mode, I switched to full alert logging:

```bash
sudo snort -c /etc/snort/snort.conf -Q --daq afpacket -i eth0:eth1 -A full -l /var/log/snort
```

The exercise required the traffic to remain blocked for at least one minute before the flag appeared. I completed this step and obtained the second flag. The mission debrief also reported that I wrote to and inspected `/var/log/snort/alert`, confirming that alert records were being retained.

To inspect such alerts during future practice, I can use commands such as:

```bash
sudo tail -n 30 /var/log/snort/alert
```

For a specific rule, I can filter the alert file using its Snort rule identifier:

```bash
sudo grep '1000002' /var/log/snort/alert
```

These are useful follow-up commands for revision. I am not claiming that I automated the checks or developed a monitoring script during this room. The important practical distinction is that live console messages show what Snort is doing at that moment, while persisted alert files allow later review and verification.

## Troubleshooting Lessons

My main configuration difficulty involved AFPacket initialization. The initial single-interface command failed even though the Snort configuration file and rule were successfully parsed. Checking `--daq-list` and `ip -br link` helped distinguish a DAQ/interface problem from a syntax error in the detection rule.

I also learned why successful rule loading, active packet dropping, and alert-file creation should be checked separately. A rule can load without ever matching traffic. A console alert can show a match, but the IPS must be positioned inline to enforce the drop. Full logging provides records of the activity but is not a substitute for verifying prevention.

The second scenario gave particularly clear evidence because Snort printed `[Drop]` repeatedly for TCP connections toward port 4444. I did not need to infer blocking solely from the eventual flag.

## Conclusion

This room helped me connect packet inspection, Snort rule syntax, inline IPS configuration, and alert logging into one practical defensive workflow. I examined repeated SSH connections and suspicious outbound traffic, wrote targeted `drop` rules, resolved an AFPacket interface initialization error, and confirmed packet drops in console output before using full logging to complete the lab.

The most important lesson is that network defense requires verification at each stage. Identifying an unusual port is not enough to prove an attack, loading a rule is not enough to prove it matched, and enabling logging is not enough to prove traffic was blocked. In this controlled exercise, I could connect the observed traffic, the matching rule, the `[Drop]` output, and the completed containment objective.
