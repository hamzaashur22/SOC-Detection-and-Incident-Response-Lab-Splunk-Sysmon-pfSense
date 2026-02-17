# SOC Detection & Incident Response Lab

## Overview 
This repository documents an end-to-end SOC workflow in a home lab. A Windows host is intentionally exposed via RDP, an attacker performs recon and unauthorized access activity, Splunk detects suspicious behavior from Sysmon/Windows Security telemetry, the analyst investigates and confirms the intrusion path, then containment is enforced using pfSense quarantine rules. The lab concludes with recovery and post-incident actions, including a written incident response report coupled with lessons learned and hardening changes. The lab ends off with adding automation-focused improvements (alert-driven scripts/webhooks) to demonstrate the full SOC lifecycle from detection to response to continuous improvement. 

## What this lab demonstrates (SOC skills)
- Log ingestion validation (endpoint telemetry reliably arriving in Splunk)
- Detection engineering (alerts and dashboards built from Sysmon and Windows Security events)
- Triage workflow (alert → pivots → evidence-based conclusion)
- Containment (pfSense quarantine alias + isolation rules)
- Recovery and hardening (Reverting to known good snapshot, closing exposure, lockout policy, password resets, blocking attacker)

## Lab Architecture 
- Kali Linux: attacker host 
-  Windows Victim: endpoint
- Splunk Server: SIEM 
- pfSense: local router/firewall

## Diagram

<img width="620" height="667" alt="Screenshot 2026-02-08 111142" src="https://github.com/user-attachments/assets/d0b9a929-31aa-4502-90ba-2305ce6d098c" />  

## Attack Scenario 
1. Recon: attacker identifies exposed RDP (3389)
   <img width="868" height="645" alt="Screenshot 2026-02-08 113020" src="https://github.com/user-attachments/assets/7e3bdc66-c242-4e0a-a28c-9102b8b90e22" />

2. Initial access activity: Attacker taking advantage of the misconfiguration  brute-forces the vulnerable instance
   <img width="1213" height="703" alt="Screenshot 2026-02-08 113202" src="https://github.com/user-attachments/assets/8c84fc87-53f5-4a47-aac3-32135750eb99" />

3. Persistence/privilege: attacker creates a user and elevates privileges
   <img width="895" height="600" alt="Screenshot 2026-02-08 113624" src="https://github.com/user-attachments/assets/2cdd1ae1-d46f-491f-b927-4169cadb94f3" />

4. Detection: Splunk alert triggers. 
   <img width="1008" height="649" alt="Screenshot 2026-02-08 113746" src="https://github.com/user-attachments/assets/9c48f916-02c7-477f-ae76-52bbd35035a0" />

5. Analyst commences triage and notices that the user that added the new user was a previously inactive account. 
<img width="1197" height="648" alt="Screenshot 2026-02-08 122009" src="https://github.com/user-attachments/assets/df554517-5cb9-48f6-ac2f-3544303a322f" />

6.  Analyst opens a incident case and begins case notes.
<img width="812" height="595" alt="Screenshot 2026-02-08 121633" src="https://github.com/user-attachments/assets/34b937b8-71ad-4903-a2d7-42037c208989" />

7. Using the previously acquired information (account being used)  analyst hones in on all logs from the suspicious user during the account privilege escalation specifically failed/successful logon event codes.
    It is revealed that a suspicious foreign ip has brute forced into the rdp-victim account. Reasonable assumptions can be made that a hacker has remotely got access to the instance.
    
<img width="918" height="663" alt="Screenshot 2026-02-08 123119" src="https://github.com/user-attachments/assets/2debaf62-5b62-4e43-a85c-01135d7499ba" /> <img width="885" height="661" alt="Screenshot 2026-02-08 123218" src="https://github.com/user-attachments/assets/a54e5e8e-feef-4879-8ea6-5ce1e2abb80d" />


8. Analyst searches for network connections through sysmon EventId 3 with a focus on the foreign ip recently discovered leading them to come to the following conclusions. Attacker most likely scanned our network found a misconfigured open rdp port on our windows host and
    attempted brute forcing thus finally getting into the network.

    <img width="981" height="676" alt="Screenshot 2026-02-08 124216" src="https://github.com/user-attachments/assets/2584e54b-067a-44cf-a435-79bc4debba8f" />


9. Updated live case notes

<img width="700" height="728" alt="Screenshot 2026-02-08 145942" src="https://github.com/user-attachments/assets/a47aedec-b929-4097-8287-9afa753fd626" />

  
10.  Containment: After getting the full picture and timeline analyst moves to isolate the host using the pfsense firewall to completely block off any outbound connection.

<img width="1074" height="665" alt="Screenshot 2026-02-08 124624" src="https://github.com/user-attachments/assets/8433ca13-1242-45c4-803c-8d29600542ac" />

    
11. Recovery: Once the host has been completely isolated analyst goes in and reverts the windows instance back to a known good snapshot

    <img width="864" height="650" alt="Screenshot 2026-02-08 124751" src="https://github.com/user-attachments/assets/593b23ec-bf3a-48fd-a342-b55fd6b091c9" />

    
12. Recovery/Hardening: Analyst resets all credentials on affected host  + Closes the prviously misconfigured RDP port + Added a lockout policy to prevent future brute force attacks

    
  <img width="522" height="327" alt="Screenshot 2026-02-08 125233" src="https://github.com/user-attachments/assets/3b3c5069-71ea-4e76-93c0-1b562cbff8b3" /> <img width="351" height="394" alt="Screenshot 2026-02-08 125250" src="https://github.com/user-attachments/assets/c47c9317-06a0-4974-bcff-7b2ce8ff0ae5" />
<img width="406" height="385" alt="Screenshot 2026-02-08 125336" src="https://github.com/user-attachments/assets/ea19a799-8012-4908-8461-0eb00d6c86c6" />


13. Improvements: Analyst creates a Recon / Pre-Compromise dashboard that allows the user to detect port scan activity to our Window’s host through connection count

    <img width="770" height="521" alt="Screenshot 2026-02-08 125820" src="https://github.com/user-attachments/assets/b3844949-f090-4b87-a67b-00e56c253d32" />

    The splunk query I used to create this dashboard is as follows:
    ```
    host="SOC-VICTIM" index=sysmon EventCode=3 |
    stats dc(DestinationPort) as unique_ports values(DestinationPort) as ports by SourceIp |
    sort -unique_ports
    ```
14. Alert/Automation:  Created an alert that triggers a webhook once an unauthorized rdp connection is made. Webhook automatically isolates host. More details on webhook configuration here: [Alert setup](docs/docs/setup.md)

 
<img width="1553" height="627" alt="Screenshot 2026-02-08 143900" src="https://github.com/user-attachments/assets/ab7c3654-adf5-47fc-91a7-8dc55465a259" />  

Splunk query used to create the alert is as follows: 
```
host="SOC-VICTIM" index=sysmon EventCode=3 DestinationPort=3389
| where SourceIp!="192.168.5.12"
| table _time host SourceIp DestinationIp DestinationPort Image
| sort - _time
```


15. Incident Response Report
   [IR Report](docs/ir-report.md)






