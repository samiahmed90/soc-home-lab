# SQL Injection Detection with Wazuh

## Objective
Simulate a SQL injection attack from Kali Linux against 
an Apache web server and detect it using Wazuh SIEM 
in real time.

## Environment
- **Attacker:** Kali Linux (192.168.68.112)
- **Victim:** Ubuntu Server with Apache2 (192.168.68.111)
- **SIEM:** Wazuh Manager (192.168.68.110)
- **Log Source:** /var/log/apache2/access.log

## Lab Setup
1. Installed Apache2 on Ubuntu agent
2. Configured Wazuh agent to monitor Apache access logs:
```xml
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>
```
3. Restarted Wazuh agent to apply configuration
4. Verified Kali could reach Apache server via curl

## Attack Simulation
SQL injection attack launched from Kali Linux using 
URL-encoded payload:

```bash
curl -XGET "http://192.168.68.111/%27select%20*%20from%20users"
```

**Why URL encoding:**
The Wazuh SQL injection rule (0245-web_rules.xml) matches 
URL-encoded patterns like `%27` (single quote) which is 
the classic SQL injection signature. Plain text SELECT 
statements do not trigger the rule.

## Wazuh Alerts Triggered

| Timestamp | Rule ID | Description | Level |
|-----------|---------|-------------|-------|
| 23:53:12 | 31103 | SQL injection attempt | 7 |
| 23:52:54 | 31106 | Web attack returned code 200 (success) | 6 |
| 23:52:32 | 31164 | SQL injection attempt | 6 |

## Alert Details
- **Agent:** samiahmed90 (192.168.68.111)
- **Attacker IP:** 192.168.68.112 (Kali)
- **Protocol:** GET
- **URL:** /%27select%20*%20from%20users
- **Log source:** /var/log/apache2/access.log
- **GDPR mapping:** IV_35.7.d
- **Rule fired:** 3 times

## Key Finding
Rule 31106 fired with description "A web attack returned 
code 200 (success)" indicating the web server responded 
successfully to the malicious request. While the SQL 
injection itself did not execute (no database backend), 
the request reached the server and received a response — 
confirming the attack surface exists and the server is 
reachable by the attacker.

## Severity
**High** — SQL injection attempt detected and confirmed 
reachable. Database backend would be at risk in a 
production environment.

## MITRE ATT&CK Mapping
| Tactic | Technique ID | Technique Name |
|--------|-------------|----------------|
| Initial Access | T1190 | Exploit Public-Facing Application |
| Discovery | T1083 | File and Directory Discovery |

## Recommended Response
1. Block attacker IP 192.168.68.112 at firewall
2. Implement Web Application Firewall (WAF) to filter
   SQL injection patterns before reaching the server
3. Use parameterized queries in all database calls
   to prevent SQL injection execution
4. Enable Wazuh active response to auto-block IPs
   triggering SQL injection rules
5. Review all web application input validation
6. Conduct full web application penetration test

## Screenshots
[Screenshot 1 — Kali curl command reaching Apache server]
<img width="1284" height="968" alt="Screenshot 2026-07-18 235209" src="https://github.com/user-attachments/assets/42c9ae99-96a1-470b-957e-8f164cee104d" />

[Screenshot 2 — SQL injection curl command with 404 response]
<img width="1284" height="968" alt="Screenshot 2026-07-18 235617" src="https://github.com/user-attachments/assets/ebbdbd4b-56b6-445b-a2f1-fd8b27ac54a6" />

[Screenshot 3 — Wazuh Threat Hunting showing 3 alerts]
<img width="1420" height="751" alt="preview (1)" src="https://github.com/user-attachments/assets/f62f0e73-7ab4-4d96-a3f0-ae757161d750" />

[Screenshot 4 — Alert document details showing attack info]
<img width="1420" height="770" alt="preview" src="https://github.com/user-attachments/assets/7358f085-ef67-4bc8-99e5-a9f9bac6c76f" />

