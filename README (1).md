# Network Reconnaissance, Traffic Analysis & Exploitation in a Segmented Lab Environment

**Course:** Network Security (IB433C), Stockholm University — Spring 2026
**Collaborators:** Completed with Radi Shafiq and Rakibul Islam Rahat
**Environment:** Isolated university lab network (10.11.202.0/23), Kali Linux attack host, Windows 10 and Linux targets, Metasploitable2

## Overview

This project walks through a condensed attack lifecycle in a controlled lab: discovering hosts and services, capturing and interpreting live traffic, intercepting cleartext credentials from a man-in-the-middle position, exploiting a vulnerable web service for remote code execution, and running an automated vulnerability scan to compare against manual findings.

## Part 1 — Host Discovery & Service Enumeration (Nmap / Zenmap)

Using Zenmap's ping-scan profile across `10.11.202.0/23`, we identified live hosts on the network before running targeted scans against three systems:

| Host | OS | Open Ports | Notes |
|---|---|---|---|
| 10.11.202.12 | Windows 10 | 135 (MSRPC), 139 (NetBIOS), 445 (SMB), 3389 (RDP) | Typical Windows networking footprint |
| 10.11.202.19 | Linux | 22 (SSH) only | Minimal attack surface |
| 10.11.202.13 | Metasploitable2 | FTP, SSH, Telnet, SMTP, DNS, HTTP, SMB, MySQL, PostgreSQL, VNC, and more | Deliberately misconfigured for training |

**Takeaway:** attack surface tracks directly with exposed services — the Linux host running only SSH had a materially smaller attack surface than Metasploitable's sprawling service list. Host discovery before port scanning also matters operationally: it narrows scan scope and reduces the noise (and detection risk) of a full-range scan.

## Part 2 — Traffic Capture & Protocol Analysis (tcpdump, Wireshark, Snort)

Captured live traffic on `eth0` with `tcpdump` and Wireshark while generating ordinary browsing activity. Traffic broke down into a few recognizable categories:

- **ARP** — address resolution on the local segment
- **ICMP** — connectivity checks and "destination unreachable" responses for ports with no listening service
- **TLS / QUIC** — encrypted application traffic (modern HTTPS), where payload content isn't visible but metadata (endpoints, timing, volume) still is

Running Snort in packet-logger mode surfaced the same traffic with more structure. Switching to IDS mode (loading the full local ruleset, ~4,000 rules) showed the difference between raw capture and *rule-driven* analysis: Snort adds detection and alerting on top of raw packet capture, so instead of manually reviewing every packet, alerts surface only the traffic that matches a known-bad pattern — faster, but only as good as the ruleset behind it.

## Part 3 — Cleartext Credential Interception & MITM (dsniff, Ettercap)

With `dsniff` listening on `eth0`, we authenticated to Metasploitable's FTP and Telnet services and watched the credentials appear in plaintext in the dsniff output — confirming neither protocol encrypts authentication traffic.

We then escalated to an active man-in-the-middle position:

1. Enabled IP forwarding on the Kali host (`sysctl -w net.ipv4.ip_forward=1`)
2. Ran an ARP-spoofing attack with Ettercap, poisoning the ARP caches of the Windows client and the Metasploitable server so traffic between them routed through the Kali box
3. Verified the poisoning by checking the Windows ARP table (the Kali MAC now resolved for the target IP)
4. Connected from Windows to Metasploitable via Telnet, and captured the full session — credentials and every command typed — via `dsniff`

**Why it matters:** this is the hands-on version of "Telnet is insecure." An attacker who can get onto the same broadcast segment doesn't need to break encryption that doesn't exist — they just need to be in the traffic path. SSH would have prevented this outright, since session content would have been encrypted before it ever hit the wire.

## Part 4 — Exploitation (Metasploit)

Target: a Windows host running **Simple Web Server 2.2 rc1**, a known-vulnerable application used for training, exposing a buffer overflow in its connection handling.

- Confirmed the open port with a targeted Nmap scan
- In Metasploit, located the matching module (`exploit/windows/http/sws_connection_bof`), configured `RHOSTS`, `RPORT`, `LHOST`, and `LPORT`, and set the payload to `windows/meterpreter/reverse_tcp`
- Executed the exploit and obtained a Meterpreter session
- Ran post-exploitation reconnaissance (`sysinfo`, `getuid`, `ipconfig`) to confirm system details, privilege level, and network configuration
- Demonstrated impact by writing a proof file to the target's desktop

**Takeaway:** a vulnerability scan can flag a service as outdated without confirming it's actually exploitable — the buffer overflow here is the difference between "theoretically risky" and "remote code execution achieved." Meterpreter's reverse-connection design is also worth noting defensively: because the *target* initiates the outbound connection back to the attacker, naive inbound-only firewall rules don't stop it.

## Part 5 — Vulnerability Scanning (GVM/OpenVAS)

Configured GVM/OpenVAS against the Metasploitable target and ran a full scan. Results included multiple critical and high-severity findings, consistent with Metasploitable's intentionally outdated service versions — a useful sanity check against the manual findings from Parts 1 and 4.

## Skills Demonstrated

`Nmap` `Zenmap` `Wireshark` `tcpdump` `Snort (sniffer / logger / IDS modes)` `Ettercap (ARP spoofing)` `dsniff` `Metasploit Framework` `Meterpreter post-exploitation` `GVM / OpenVAS` `protocol security analysis` `MITM attack execution & defense`

## Reflection

The through-line across all five parts is that security posture is a function of exposed surface and unencrypted trust, not any single control. A minimal, patched service list (the Linux SSH-only host) resists most of what worked here; a sprawling, outdated one (Metasploitable) falls to nearly all of it. Automated scanning is a good first pass, but it's not a substitute for validating exploitability, and it won't catch a MITM position or explain why a given signature does or doesn't fire — that still takes manual analysis.
