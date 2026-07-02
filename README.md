# Network Security Portfolio

**Akshat Rawat** · MSc Information Security, Stockholm University
📫 *https://www.linkedin.com/in/akshat-rawat-2001/*

Hands-on network security coursework from my Master's program, written up as case studies rather than raw assignment answers. Covers reconnaissance, traffic analysis, intrusion detection, exploitation[...]

## Projects

### [01 — Network Reconnaissance, Traffic Analysis & Exploitation](./Network%20Reconnaissance%2C%20Traffic%20Analysis%20%26%20Exploitation%20in%20a%20Segmented%20Lab%20Environment.md)
A two-part sequence covering the full loop from host discovery to exploitation: Nmap/Zenmap scanning, live traffic capture with Wireshark/tcpdump/Snort, ARP-spoofing MITM with credential interception,[...]
**Tools:** Nmap, Zenmap, Wireshark, tcpdump, Snort, Ettercap, dsniff, Metasploit, Meterpreter, GVM/OpenVAS

### [02 — IDS Evasion & the Limits of Signature-Based Detection](./IDS%20Evasion%20%26%20the%20Limits%20of%20Signature-Based%20Detection.md)
Systematic adversarial testing of a Snort IDS rule — probing which traffic modifications (port changes, encoding, headers, timing) evade detection and why, then translating the findings into concret[...]
**Tools:** Snort, netcat, tcpdump

### [03 — Access Control Misconfiguration Assessment](./03-access-control-assessment)
An authorized assessment that uncovered a horizontal privilege escalation vulnerability in an internal Git hosting service — any authenticated user could push to any repository. Covers root cause an[...]
**Tools:** Git, manual enumeration

## About

I'm an MSc Information Security student at Stockholm University (DSV, Sweden), focused on hands-on security testing and applied defense. These projects were completed as part of coursework in Network [...]

## Scope note

All testing here was carried out in an isolated university lab network against systems provided for coursework, or as part of an explicitly authorized in-course security assessment. Nothing here targe[...]
