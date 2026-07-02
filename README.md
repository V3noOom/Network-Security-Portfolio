# Network Security Portfolio

**Akshat Rawat** · MSc Information Security, Stockholm University
📫 *https://www.linkedin.com/in/akshat-rawat-2001/*

Hands-on network security coursework from my Master's program, written up as case studies rather than raw assignment answers. Covers reconnaissance, traffic analysis, intrusion detection, exploitation, and access-control auditing in an isolated lab environment.

## Projects

### [01 — Network Reconnaissance, Traffic Analysis & Exploitation](./Network Reconncaissance, Traffic Analysis & Exploitation)
A two-part sequence covering the full loop from host discovery to exploitation: Nmap/Zenmap scanning, live traffic capture with Wireshark/tcpdump/Snort, ARP-spoofing MITM with credential interception, exploitation of a vulnerable web service via Metasploit, and vulnerability scanning with GVM/OpenVAS.
**Tools:** Nmap, Zenmap, Wireshark, tcpdump, Snort, Ettercap, dsniff, Metasploit, Meterpreter, GVM/OpenVAS

### [02 — IDS Evasion & the Limits of Signature-Based Detection](./02-ids-evasion-and-detection)
Systematic adversarial testing of a Snort IDS rule — probing which traffic modifications (port changes, encoding, headers, timing) evade detection and why, then translating the findings into concrete recommendations for more robust rule design.
**Tools:** Snort, netcat, tcpdump

### [03 — Access Control Misconfiguration Assessment](./03-access-control-assessment)
An authorized assessment that uncovered a horizontal privilege escalation vulnerability in an internal Git hosting service — any authenticated user could push to any repository. Covers root cause analysis, OWASP/CWE classification, impact assessment, and remediation guidance. *(Identifying details of the system and other individuals have been generalized — see the note at the top of the write-up.)*
**Tools:** Git, manual enumeration

## About

I'm an MSc Information Security student at Stockholm University (DSV, Sweden), focused on hands-on security testing and applied defense. These projects were completed as part of coursework in Network Security (IB433C).

## Scope note

All testing here was carried out in an isolated university lab network against systems provided for coursework, or as part of an explicitly authorized in-course security assessment. Nothing here targets production systems.
