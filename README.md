# SOC-Incident-Response-Portfolio-End-to-End-Threat-Detection-Mitigation
Executive Summary
This portfolio showcases a series of hands-on Security Operations Center (SOC) labs focused on the NIST Incident Response Life Cycle. Across five distinct cases, I demonstrate the ability to detect, analyze, and contain threats in both Windows and Linux environments using industry-standard tools and native OS utilities.

Lab Summaries
Case 1: Windows RDP Brute Force Attack Investigated a targeted dictionary attack by correlating Security Event ID 4625 and RDP Operational Event ID 1149 to identify the attacker's IP and implemented a host-based firewall block.
Case 2: Linux Suspicious Process IR Identified and surgically terminated a malicious background process (fake_backup.sh) mimicking legitimate activity using PID analysis and SIGKILL signals.
Case 3: Linux Persistence via Malicious Cron Job Neutralized an automated persistence mechanism by identifying unauthorized scheduled tasks in the system crontab and verifying eradication through service audits.
Case 4: Suspicious PowerShell Activity Utilized advanced GPO logging and Event ID 4103 to detect "Living off the Land" (LotL) techniques targeting the sensitive Windows hosts file.
Case 5: Network Beaconing & C2 Mitigation Correlated network sockets to specific processes to identify and block automated Command and Control (C2) communication using tcpdump, lsof, and iptables.

Technical Skills Demonstrated
Log Forensics & Analysis: Proficient in Windows Event Viewer (Security, PowerShell, and Terminal Services logs) and Linux monitoring utilities (ps, netstat, lsof, tcpdump).
Advanced System Hardening: Configuring Group Policy Objects (GPOs) for deep telemetry and utilizing host-based firewalls (Windows Defender & iptables) for network isolation.
Threat Detection & Containment: Expert in identifying malicious persistence (Cron jobs), process manipulation, and network beaconing, followed by rapid containment via process termination and IP blocking.
Forensic Evidence Collection: Preserving critical data through command-line forensics (/proc/cmdline) and capturing live network traffic into .pcap files for deep-packet analysis.
