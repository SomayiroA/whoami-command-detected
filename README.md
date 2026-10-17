# Event ID 118 - Whoami Command Detected in Request Body


## Short Overview of the Investigation 🔬

This investigation examines a high-severity LetsDefend alert triggered by an HTTP POST request containing the whoami command. Further investigation revealed additional operating-system commands sent to the web server, including attempts to read sensitive system files. Endpoint telemetry confirmed command execution, indicating a successful command injection attack.

*Investigation verdict*: True Positive - Successful command injection and remote code execution (RCE).


## Alert Summary and Timeline ⌛ 

### Alert Summary

| Field | Details |
| ----- | ------- |
| Platform | LetsDefend |
| Event ID | 118 |
| Rule | SOC168 - Whoami Command Detected in Request Body |
| Severity | High|
| Event Date | February 28, 2022 |
| Event Time | 04:12 AM |
| Hostname | WebServer1004 |
| Source IP | 61.177.172.87 |
| Destination IP | 172.16.17.16 |
| HTTP Method | GET |
| Requested URL | https://172.16.17.16/video/ |
| User-Agent | Internet Explorer 6.0 (legacy user-agent string) |
| Device Action | Allowed |
| Attack Type | OS Command Injection / RCE |

### Timeline

- *04:12 AM*: The alert was generated after the request body contained whoami.
- *Log investigation*: Five related POST requests from the same source IP were identified.
- *Command analysis*: Requests contained commands such as ls, whoami, uname, cat /etc/passwd, and cat /etc/shadow.
- *Endpoint investigation*: Command history and process information confirmed that the commands had been executed.
- *Final assessment*: The attack was classified as a true positive, requiring containment and Tier 2 escalation.


## Investigation Steps 🕵️‍♂️ 

### 1. Alert Triage 
I reviewed the alert details, source and destination IP addresses, requested URL, HTTP method, and device action. The firewall allowed the request, meaning the suspicious traffic was not blocked at that layer.

### 2. Log Management and Traffic Analysis 
I then filtered network logs using the source IP address 61.177.172.87. Five POST requests to the same endpoint were identified, containing the following parameters:

- ?c=ls
- ?c=whoami
- ?c=uname
- ?c=cat /etc/passwd
- ?c=cat /etc/shadow

These commands indicated system enumeration and attempts to access sensitive account information. The requests returned HTTP 200 responses with response sizes exceeding 900 bytes.

### 3. Threat Intelligence Investigation
I investigated the source IP using threat intelligence resources. The IP had been associated with suspicious and malicious activity. This supported the assessment that the requests were not ordinary application traffic.

IP reputation alone is not proof of an attack, but in this case it aligned with the command-injection evidence.

### 4. Endpoint Investigation
I reviewed the endpoint telemetry for WebServer1004, focusing on command history and process information.

The investigation confirmed that commands including whoami, uname, and cat had executed. The output associated with /etc/passwd and /etc/shadow indicated that sensitive system information had been accessed.

*This endpoint evidence was critical*: a HTTP 200 response by itself would not prove that operating-system commands had executed.

### 5. Attack Classification and Escalation
The combination of malicious HTTP requests, command execution evidence, and access to sensitive files confirmed a successful command injection attack.

My response was to classify the alert as a True Positive, contain the affected server, preserve evidence, and escalate to Tier 2 for further incident investigation.


## My Findings, Conclusions and Response

### Findings 
- An external source sent multiple POST requests containing operating-system commands.
- The firewall permitted the requests.
- HTTP responses returned status 200 with substantial response sizes.
- Endpoint telemetry confirmed command execution.
- Commands attempted to enumerate the system and access sensitive account files.
- The activity indicated successful command injection and remote code execution.

### Conclusion
This was a True Positive successful OS command injection leading to RCE. The attacker was able to execute system commands through the web application and access sensitive system information. The incident was contained and escalated to a Tier 2 Analyst.

### My Response
1. *The affected server was contained*: I isolated WebServer1004 according to incident response procedures while preserving evidence.
2. *The malicious traffic was blocked*: The appropriate network controls to restrict the identified source IP was applied.
3. *Forensic evidence was preserved*: I retained web, firewall, endpoint, process, and command-history logs.
4. *I investigated the scope*: Checked for persistence, credential exposure, data exfiltration, lateral movement, and additional compromised systems.
5. *Remediated the vulnerability*: I removed unsafe command execution from the web application and validated user input. Prefered safe APIs over shell execution.
6. *Protected credentials*: I assessed the exposure of account information and credential hashes, and reset affected credentials where necessary.
7. *Escalated to Tier 2*: Requested deeper investigation and coordinated containment and recovery.


## Screenshots 📷 

<img width="593" height="296" alt="image" src="https://github.com/user-attachments/assets/0d944ff0-fb08-4adb-b25b-91a1d920eb01" />

<img width="591" height="289" alt="image" src="https://github.com/user-attachments/assets/622eb732-f843-41e9-a9cb-a2a33b2b6002" />

<img width="566" height="290" alt="image" src="https://github.com/user-attachments/assets/a55f868b-2f20-4a7a-9503-c5adf1191dda" />

<img width="566" height="124" alt="image" src="https://github.com/user-attachments/assets/1bf3d4fd-6171-4f58-bde8-716144663628" />


## Lessons Learned ✍️ 

- *Investigate beyond the initial alert*: The whoami command was only the first indicator. Related requests revealed a broader sequence of malicious activity.
- *Correlate network and endpoint evidence*: HTTP logs showed what the attacker sent, while endpoint telemetry confirmed what executed.
- *Do not rely on HTTP status codes alone*: A 200 response does not independently prove command execution; response content and endpoint evidence provide stronger confirmation.
- *Recognize sensitive file access*: Attempts to read /etc/passwd and /etc/shadow can indicate account enumeration and credential exposure.
- *Contain and escalate confirmed compromises*: Successful command execution and access to sensitive system files warrant urgent incident response and Tier 2 escalation.

