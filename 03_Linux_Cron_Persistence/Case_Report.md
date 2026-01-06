Overview & Scenario
•	The Scenario: You are a SOC Analyst investigating a Linux server that seems to be re-infecting itself. You have previously killed a suspicious backup script, but it keeps reappearing.
•	The Goal: To identify the "Persistence" mechanism (Cron Job), analyze the malicious script, and permanently eradicate the threat.
________________________________________
Phase 1: Attack Simulation (Setting Persistence)
Machine: Kali Linux VM
Terminal: Root Terminal (Use sudo su to match your screenshots).
Step 1: Create the Malicious Payload
We create a script in the /tmp directory that mimics an attacker logging data.
•	Command: ```bash
echo "echo 'Attacker script running at $(date)' >> /tmp/malicious.log" > /tmp/malicious.sh
chmod +x /tmp/malicious.sh
•	Explanation: This creates a script that appends a timestamp to a log file every time it runs.
Step 2: Schedule the Cron Job
This is the "Persistence" step. We tell the system to run the script every minute.
•	Command: ```bash
(crontab -l 2>/dev/null; echo "* * * * * /tmp/malicious.sh") | crontab -
•	Explanation: The * * * * * notation represents every minute of every hour, day, month, and week. This ensures that even if you kill the process, it will start again in less than 60 seconds.
________________________________________
Phase 2: Detection & Analysis (The SOC Analyst)
Machine: Kali Linux VM
Terminal: Same Terminal.
Step 3: Identify Unauthorized Scheduled Tasks
The analyst checks the user's crontab for suspicious entries.
•	Command: crontab -l
•	Sample Output: > * * * * * /tmp/malicious.sh
•	Analysis: Finding a script running every minute from the /tmp directory is a major indicator of compromise (IoC).
Step 4: Verify the Malicious Activity
Check if the script is actually doing what it claims by reading its log output.
•	Command: cat /tmp/malicious.log
•	Sample Output: > Attacker script running at Mon Jan 5 12:16:01 EST 2026
________________________________________
Phase 3: Containment & Eradication
Machine: Kali Linux VM
Terminal: Same Terminal.
Step 5: Remove the Persistence (Containment)
You must stop the "clock" before you delete the file, or the system might try to run a non-existent file and throw errors.
•	Command: crontab -r
•	Explanation: This command removes the entire crontab for the root user, effectively "turning off" the attacker's scheduler.
Step 6: Eradicate the Files
Permanently remove the script and the logs from the disk.
•	Command: rm /tmp/malicious.sh /tmp/malicious.log
Step 7: Final Audit & System Recovery
Refresh the service and verify no tasks remain.
•	Command 1: systemctl restart cron
•	Command 2: crontab -l
•	Output: no crontab for root
________________________________________
SOC Lab Case 3 Summary of Findings
Phase	Action	Technical Finding / Explanation
Detection	crontab -l	Identified unauthorized persistence via a 1-minute cron task.
Analysis	cat /tmp/malicious.sh	Confirmed the script was executing from a temporary directory (/tmp).
Containment	crontab -r	Effectively neutralized the persistence mechanism by clearing the crontab.
Eradication	rm command	Removed malicious binaries and audit logs from the host.



