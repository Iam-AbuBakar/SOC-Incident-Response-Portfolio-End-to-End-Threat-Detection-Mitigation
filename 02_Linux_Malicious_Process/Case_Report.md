Phase 1: Attack Simulation
Machine: Kali Linux VM
Terminal: Root Terminal (Open a terminal and type sudo su to ensure you have the necessary privileges).
•	Step 1: Create the Lab Directory
o	Command: mkdir script_lab && cd script_lab
o	Explanation: You are creating a specific folder to keep your lab files organized and isolated.
•	Step 2: Create the "Malicious" Script
o	Command: nano fake_backup.sh
o	Content:
Bash
#!/bin/bash
echo "Simulating a backup operation..."
sleep 60
o	Explanation: The sleep 60 command mimics a long-running suspicious process, such as a miner or a backdoor, giving you time to "detect" it while it is active.
•	Step 3: Execution
o	Command 1: chmod +x fake_backup.sh
o	Command 2: ./fake_backup.sh &
o	Explanation: The & symbol is crucial—it runs the script in the background, allowing you to continue using the same terminal window for the investigation.
o	Sample Output: [1] 2673 (where 2673 is your unique Process ID).
________________________________________
Phase 2: Detection & Analysis
Machine: Kali Linux VM
Terminal: Same Root Terminal.
•	Step 4: Identify the Process
o	Command: ps aux | grep fake_backup.sh
o	Explanation: ps aux lists all running processes, and grep filters for your specific script.
o	Sample Output: root 2673 0.0 0.1 12856 2252 pts/0 S 10:00 0:00 /bin/bash ./fake_backup.sh.
o	Finding: You have successfully detected a script running under root privileges with PID 2673.
________________________________________
Phase 3: Containment & Eradication
Machine: Kali Linux VM
Terminal: Same Root Terminal.
•	Step 5: Kill the Process (Containment)
o	Command: kill -9 2673 (Replace 2673 with your actual PID)
o	Explanation: The -9 flag (SIGKILL) forces the process to stop immediately.
•	Step 6: Handle Expiration (Verification)
o	Finding: If you get a "no such process" error, the sleep 60 timer ended. Re-run ./fake_backup.sh & to get a new PID (e.g., 5222) and then run kill -9 5222.
o	Sample Output: [1] + killed ./fake_backup.sh.
•	Step 7: Delete the Source (Eradication)
o	Command: rm fake_backup.sh
o	Explanation: This removes the malware from the disk to prevent it from being restarted.
________________________________________
Phase 4: Final Audit
Machine: Kali Linux VM
Terminal: Same Root Terminal.
•	Step 8: Verification
o	Command: ls
o	Command: ps aux | grep fake_backup.sh
o	Finding: The directory should be empty, and the process search should only return your own grep command, proving the system is clean.
Summary of Incident Response Steps
IR Phase	Action Taken	Tool/Command
Detection	Identified root execution of fake_backup.sh	ps aux
Analysis	Pinpointed PID 5222 for surgical termination	grep
Containment	Forcefully stopped the malicious "payload"	kill -9
Eradication	Permanently removed source file to break persistence	rm
