1. Executive Summary
This lab simulates a "Living off the Land" (LotL) attack scenario. In this incident, an attacker utilizes legitimate system tools (PowerShell and Notepad) to access and potentially modify the critical Windows hosts file. By enabling advanced logging, we successfully identified the unauthorized activity via Event ID 4103 and implemented immediate containment through host isolation.
________________________________________
2. Scenario Overview
•	The Threat: An attacker has gained low-level access to a Windows Server 2022 VM.
•	The Objective: Modify the system hosts file to redirect legitimate network traffic (e.g., banking or corporate portals) to a malicious IP address.
•	The Strategy: Use PowerShell to launch Notepad to bypass standard security alerts that might trigger on unknown third-party binaries.
________________________________________
3. Phase 1: Preparation (Advanced Logging)
Machine: Windows Server 2022
Action: Configure Group Policy (GPO) to capture deep telemetry of PowerShell execution.
•	Path: Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell.
•	Configurations Enabled:
o	Module Logging: Records pipeline execution details for specific modules.
o	Script Block Logging: Records the full code block of executed scripts, even if obfuscated.1
o	Transcription: Creates a full text record of the PowerShell session input and output.2
•	Technical Reason: Default Windows logging is insufficient to see the specific arguments (like file paths) used in a command.
________________________________________
4. Phase 2: Attack Simulation
Machine: Windows Server 2022
Terminal: PowerShell (Administrator)
•	Command Executed:
PowerShell
Start-Process notepad.exe -ArgumentList "C:\Windows\System32\drivers\etc\hosts"
•	Command Breakdown:
o	Start-Process: The cmdlet used to launch an application.
o	notepad.exe: The legitimate binary being used to open the file.
o	-ArgumentList: Specifies the target file path.
o	"C:\Windows\System32\drivers\etc\hosts": The sensitive system file targeted for DNS hijacking.
________________________________________
5. Phase 3: Detection & Analysis
Tool: Windows Event Viewer
Step 1: Event Identification
•	Log Path: Applications and Services Logs > Microsoft > Windows > PowerShell > Operational.
•	Target Event ID: 4103 (Executing Pipeline).
Step 2: Evidence Analysis (Output Explanation)
Examining the output in image_67756a.png, we find:
•	Event ID 4103: Confirms that a PowerShell pipeline was executed.
•	Parameter Binding: Under the "Details" tab, the ArgumentList is clearly visible.
•	Findings: The logs explicitly show the hosts file path was passed as an argument to notepad.exe. This correlates directly with the suspected attack pattern.
________________________________________
6. Phase 4: Incident Response (Containment & Eradication)
Step 1: Containment (Host Isolation)
To prevent lateral movement or data exfiltration, the machine must be isolated.
•	Action: Configure an Outbound Block Rule in Windows Defender Firewall.
•	Configuration:
o	Rule Type: Custom.
o	Program: All programs.
o	Action: Block the connection.
•	Output Explanation: This acts as a manual "network kill-switch," ensuring the machine cannot communicate with the attacker's infrastructure.
Step 2: Eradication (Integrity Audit)
The analyst must verify if the attacker successfully modified the target file.
•	Evidence Source: image_001405.jpg.
•	Audit Finding: The screenshot shows the hosts file remains in its default state, containing only standard comments and the localhost loopback entry (127.0.0.1).
•	Conclusion: The threat was detected and contained before any malicious modifications were saved.
