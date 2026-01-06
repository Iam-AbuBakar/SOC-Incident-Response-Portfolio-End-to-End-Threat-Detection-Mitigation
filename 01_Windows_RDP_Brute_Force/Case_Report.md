Phase 1: Environment Setup
Goal: Create an isolated network tunnel between your machines.
•	Step 1: Network Configuration (VMware)
o	Action: Set both Windows and Kali VMs to NAT mode.
o	Explanation: This creates a private subnet (e.g., 192.168.100.x). It ensures your attack traffic stays within the lab and doesn't interfere with your actual home network.
•	Step 2: IP Identification
o	Windows: Open cmd, type ipconfig. Your IP is 192.168.100.132.
o	Kali: Open terminal, type ip addr. Your IP is 192.168.100.128.
________________________________________

Phase 2: Target Configuration (The Vulnerability)
Goal: Misconfigure the Windows Server to allow the attack to succeed.
•	Step 3: Enable RDP & Lower Security
o	Path: Settings > System > Remote Desktop.
o	Action: Toggle On. Click Advanced Settings and uncheck "Require Network Level Authentication (NLA)".
o	Explanation: NLA requires a valid login before a session even starts. By disabling it, you allow Hydra to reach the login prompt and try multiple passwords.
•	Step 4: Create the Victim Account
o	Command (PowerShell Admin): net user SOC-Test Password123 /add.
o	Permissions: Add SOC-Test to the Remote Desktop Users group in Settings.
•	Once you have both the Administrator and the SOC-Test user, here is how you handle the login and the log-checking process:
•	1. Which user should you be logged in as?
•	You should stay logged into the Windows VM as Administrator.
•	Why? Only accounts with Administrative privileges can open the Event Viewer and view Security Logs. The SOC-Test user is a "Standard User" and won't have permission to see the logs of the attack directed at them.
•	The Workflow: Keep the Windows desktop open as Administrator so you can watch the Event Viewer in real-time while Kali performs the attack on the SOC-Test account in the background.
•	________________________________________
•	2. Which user's logs should you check?
•	You aren't exactly checking a "user's log," but rather the System-wide Security Log. However, you will be looking for events mentioning the SOC-Test user.
•	Here is the step-by-step to find them:
•	Open Event Viewer: (Stay logged in as Administrator).
•	Navigate: Go to Windows Logs > Security.
•	The Attack Phase: While Hydra is running on Kali, click Refresh on the right-side pane of the Event Viewer.
•	Find the "Subject": Click on any Event ID 4625 (An account failed to log on).
•	Look at the Details: In the bottom "General" tab, look for Account Name. It should say SOC-Test.
•	Analyst Tip: In a real-world SOC, seeing multiple failed logins for the same username (SOC-Test) from the same Source IP is the "Smoking Gun" that confirms a targeted Brute Force attack.
•	________________________________________
•	3. Verification: Does the user need RDP Permission?
•	Before you start the attack in Phase 3, you need to make sure Windows actually allows the SOC-Test user to use Remote Desktop, otherwise Hydra will fail for the wrong reasons.
•	Go to Settings > System > Remote Desktop.
•	Click on "Select users that can remotely access this PC" (usually at the bottom).
•	Click Add.
•	Type SOC-Test and click Check Names, then click OK.
________________________________________
Phase 5: Attack Simulation (The Kali Side)
Goal: Execute a dictionary attack to crack the password.
•	Step 5: Create the Password List
o	Command: nano password.txt.
o	Content: Enter common passwords, ensuring Password123 is included.
•	Step 6: Launch Hydra
o	Command: hydra -l SOC-Test -P password.txt rdp://192.168.100.132 -vV.
o	Finding: Hydra will display a green success message once it finds Password123.
________________________________________
Phase 4: SOC Detection (The Investigation)
Goal: Track the attacker's fingerprints in the logs.
The "Empty IP" Problem (Troubleshooting)
In your Security Log (Event ID 4624), the "Source Network Address" showed a dash (-).
•	Why? This happens because the RDP security layer (SSL/TLS) handles the connection before the Windows Security log is triggered, causing the IP information to be "stripped" from the entry.
•	Wait! This is common when Logon Type is not Type 10 or the Security Layer is set to "Negotiate" or "SSL".
The 3 Reliable Methods to Find the Attacker IP
•	Method 1: Check Failed Events (Event ID 4625)
o	Path: Event Viewer > Windows Logs > Security.
o	Action: Find the "Audit Failure" logs created by Hydra.
o	Finding: Unlike the success log, these often capture the IP. Your log shows 192.168.100.128.
•	Method 2: Check RDP Operational Logs
o	Path: Applications and Services Logs > Microsoft > Windows > TerminalServices-RemoteConnectionManager > Operational.
o	Action: Look for Event ID 1149 ("User authentication succeeded").
o	Finding: This event explicitly logs the Source Network Address of the client.
•	Method 3: Local Session Manager
o	Path: Applications and Services Logs > Microsoft > Windows > TerminalServices-LocalSessionManager > Operational.
o	Action: Look for Event ID 21 (Session logon succeeded).
o	Finding: This also captures the remote IP that initiated the session.
Project Tip (Command Line): To see active connections in real-time, run netstat -n | find ":3389" in Command Prompt.
________________________________________
Phase 5: Incident Response (Containment & Eradication)
Goal: Block the identified IP using the host-based firewall.
Step 7: Configure the Firewall Rule
•	Path: Windows Defender Firewall with Advanced Security > Inbound Rules > New Rule.
•	Rule Type: Select Custom.
•	Protocol and Ports:
o	Protocol type: Set to TCP.
o	Local port: Set to 3389 (the RDP service port on your machine).
o	Remote port: Set to All Ports (attackers often use a random high-numbered port to connect from).
•	Scope (Understanding Local vs. Remote):
o	Local IP Address: Refers to your Windows Server VM. Select "Any IP address" to apply this to all network interfaces.
o	Remote IP Address: Refers to the Kali Attacker VM. Select "These IP addresses", click Add, and enter 192.168.100.128.
•	Action: Select Block the connection.
•	Finalize: Name the rule something clear, like Block_Kali_Brute_Force.

•	Step 8: Verification
o	Command (Kali): nmap -p 3389 192.168.100.132.
o	Finding: Port status is now Filtered, confirming the attacker is locked out
•	Verification
•	Once the rule is active, return to your Kali VM and run the following command: nmap -p 3389 192.168.100.132 The port should now show as filtered, indicating that the Windows Firewall is successfully dropping packets from the Kali IP.
