1. Executive Summary
This report details the investigation of a suspected Command and Control (C2) beaconing incident on a Linux host. Using native Linux forensic tools, we identified an infinite looping curl script communicating with a suspicious external IP address. The incident was successfully resolved through process termination and network-level isolation.
________________________________________
2. Scenario Overview
•	The Scenario: A Linux server (Kali VM) is suspected of "beaconing"—sending repeated, automated requests to a malicious external server.
•	The Objective: Detect live network anomalies, correlate them to a specific process, collect forensic evidence, and contain the threat.
•	VM Machine: Kali Linux.
•	Terminal: Root Terminal (Required for network and process-level forensics).
________________________________________
3. Phase 1: Preparation & Incident Simulation
Terminal 1 — Incident Simulation
•	Command: sudo ping -f 8.8.8.8
•	Explanation: This command initiates a "flood" ping, simulating abnormal outbound traffic and high packet rates typical of DoS behavior or malware communication.
________________________________________
4. Phase 2: Identification & Detection
Terminal 2 — Network Monitoring
•	Command: sudo tcpdump -i eth0
•	Purpose: Monitors live network traffic on the primary interface (eth0).
•	Detection Finding: Analysts observe continuous ICMP packets and repeated outbound connections to external IPs, confirming the system is generating unusual activity.
Terminal 3 — Process-to-Network Correlation
•	Command: sudo lsof -i
•	Purpose: Maps active network connections to specific Process IDs (PIDs).
•	Key Output Analysis:
o	Process: curl (PID: 30312).
o	Target: 45.13.220.98:http.
o	Status: SYN_SENT.
•	Explanation: SYN_SENT indicates the process is repeatedly attempting to connect to a suspicious IP, a strong indicator of automated beaconing.
________________________________________
5. Phase 3: Deep Analysis & Forensics
Step 5 — Process Discovery (PID Identification)
•	Command: ps aux | grep curl
•	Forensic Evidence: The output reveals an infinite loop: bash -c "while true; do curl http://45.13.220.98/ping; sleep 2; done".
•	Root Cause: This confirms automated persistence where the script re-triggers a connection every 2 seconds.
Step 6 & 7 — Evidence Preservation
•	Command Line Forensics: cat /proc/22348/cmdline (using the discovered PID).
•	Network Capture: sudo tcpdump -i eth0 host 45.13.220.98 -w incident_http_beacon.pcap.
•	Purpose: Captures the raw traffic in a .pcap file for deeper analysis in Wireshark and preserves the command-line arguments for the legal/reporting chain of custody.
________________________________________
6. Phase 4: Containment & Eradication
Step 8 — Process Termination
•	Command: sudo pkill curl
•	Explanation: This kills all instances of the malicious curl process simultaneously.
Step 9 — Network Isolation
•	Command: sudo iptables -A OUTPUT -d 45.13.220.98 -j DROP
•	Explanation: This adds a firewall rule to block all future outbound traffic to the attacker's IP, preventing reinfection if hidden scripts remain.
________________________________________
7. Phase 5: Verification (Final Audit)
Commands:
•	ps aux | grep curl
•	sudo lsof -i | grep 45.13.220.98
•	Final Finding: Zero curl processes found, zero active connections, and 0 packets captured by tcpdump.
