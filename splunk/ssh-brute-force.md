# SSH Brute Force Detection

## Objective
Investigate failed SSH login attempts on a Linux server
to identify brute force activity, targeted accounts,
attacking IP addresses, and potential account compromise.

## Environment
- **SIEM:** Splunk Enterprise 10.4
- **Log Source:** Linux Auth Log (linux_auth_logs.json)
- **Host:** LinuxUbuntuServer1
- **Log Timeframe:** March 22-23, 2025

## SPL Queries Used

**Top targeted users by failed attempts:**
index="main" sourcetype="_json" event="Failed password"
| stats count by user | sort -count

**Top attacking IPs:**
index="main" sourcetype="_json" event="Failed password"
| stats count by source_ip | sort -count

**Admin account timeline investigation:**
index="main" sourcetype="_json" user="admin"
source_ip="34.70.28.83"
| table timestamp event status | sort timestamp

## Findings

### Overview
- 3,000 total SSH authentication events detected
- 1,500 failed password attempts across 13 user accounts
- 1,500 successful logins recorded across the same period
- 13 unique source IPs identified — each targeting one 
  specific user account

### Top Targeted Accounts
| User | Failed Attempts |
|------|----------------|
| service_account | 268 |
| light_yogami | 262 |
| tony_stark | 246 |
| ryuk | 240 |
| thor | 238 |
| admin | 236 |

### Top Attacking IPs
| Source IP | Count |
|-----------|-------|
| 8.215.234.245 | 268 |
| 223.33.108.163 | 262 |
| 187.112.42.165 | 246 |
| 101.43.42.9 | 240 |
| 98.110.184.71 | 238 |
| 34.70.28.83 | 236 |

### Attack Pattern
The attack shows characteristics of a distributed brute 
force campaign — 13 unique IPs each assigned to target 
one specific account. This technique is used to avoid 
account lockout thresholds that trigger after multiple 
failures from a single IP.

### Admin Account Deep Dive
Further investigation into user "admin" from IP 
34.70.28.83 revealed 118 failed and 138 successful 
logins from the same IP across the timeframe. However 
the timeline shows a mixed pattern of failures and 
successes rather than a clean brute force to compromise 
sequence. This could indicate:
- The attacker had partial credential knowledge
- Shared IP usage between legitimate user and attacker
- Intermittent access testing

This requires further investigation to confirm malicious 
intent before escalating to confirmed compromise.

## Severity
**High** — Large scale distributed brute force attack 
confirmed across 13 accounts. Admin account activity 
from attacking IP requires further investigation.

## MITRE ATT&CK Mapping
| Tactic | Technique ID | Technique Name |
|--------|-------------|----------------|
| Credential Access | T1110.001 | Brute Force: Password Guessing |
| Initial Access | T1078 | Valid Accounts |

**T1110.001** — Attacker used automated password guessing 
across multiple accounts via SSH.

**T1078** — Successful logins recorded alongside brute 
force activity. If logins from attacking IPs are confirmed 
malicious, attacker gained access using valid credentials 
making detection significantly harder.

## Recommended Response
1. Cross-reference all successful login IPs against the
   13 identified attacking IPs to confirm compromise
2. Temporarily lock all 13 targeted accounts pending review
3. Block all 13 attacking IPs at the firewall immediately
4. Review all post-login activity from suspicious IPs
   for signs of lateral movement or data exfiltration
5. Enable fail2ban to automatically block IPs after 
   repeated failed attempts
6. Enforce SSH key-based authentication and disable 
   password login entirely
7. Rotate credentials for all affected accounts
8. Monitor for reconnection attempts from new IPs 
   using the same usernames

## Screenshots
<img width="2880" height="1524" alt="Screenshot 2026-07-05 091735" src="https://github.com/user-attachments/assets/7291aeed-e5b1-4b28-9100-2b27d6fd83e4" />
<img width="2880" height="1528" alt="Screenshot 2026-07-05 091836" src="https://github.com/user-attachments/assets/5da38f0c-434d-4c6d-9ffa-c1a1c4cea47d" />
<img width="2880" height="1532" alt="Screenshot 2026-07-05 095529" src="https://github.com/user-attachments/assets/f96a3e56-e610-4db6-b4dc-adec4b472f39" />



