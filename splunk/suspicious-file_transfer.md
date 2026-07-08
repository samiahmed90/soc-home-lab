# Suspicious FTP File Transfer - Data Exfiltration

## Objective
Investigate suspicious outbound FTP traffic to identify
unauthorized data exfiltration, compromised accounts,
and external destinations receiving stolen data.

## Environment
- **SIEM:** Splunk Enterprise 10.4
- **Log Source:** ftp_suspicious_transfer_logs.json
- **Index:** network
- **Log Timeframe:** August 2, 2025

## SPL Queries Used

**Identify FTP traffic on port 21:**
index="network" sourcetype="firewall_logs"
| where dest_port=21

**Filter large outbound transfers:**
index="network" sourcetype="firewall_logs"
| where dest_port=21 | where bytes_out > 100000

**Investigate svc_upload activity:**
index="network" sourcetype="firewall_logs"
user="svc_upload" | where dest_port="21"
| table timestamp src_ip src_port dest_ip dest_port

**Total bytes exfiltrated by user and destination:**
index="network" sourcetype="firewall_logs" dest_port=21
| stats sum(bytes_out) as total_bytes by user dest_ip
| sort -total_bytes

**Total bytes exfiltrated by svc_upload:**
index="network" sourcetype="firewall_logs"
user="svc_upload"
| stats sum(bytes_out) as total_bytes

## Findings

### Overview
- Suspicious outbound FTP transfers detected on port 21
- 4 compromised accounts identified: `backupsvc`, 
  `svc_upload`, `admin`, `system`
- 3 external destination IPs receiving stolen data
- Total data exfiltrated exceeds 1GB

### Exfiltration Breakdown
| User | Destination IP | Total Bytes | ~Size |
|------|---------------|-------------|-------|
| backupsvc | 89.45.23.77 | 558,825,605 | ~533MB |
| svc_upload | 143.110.233.10 | 95,023,507 | ~90MB |
| admin | 8.8.8.8 | 71,402,926 | ~68MB |
| admin | 13.58.97.152 | 71,257,256 | ~68MB |
| system | 8.8.8.8 | 70,484,689 | ~67MB |
| svc_upload | 8.8.8.8 | 63,604,538 | ~60MB |
| system | 143.110.233.10 | 56,927,285 | ~54MB |
| system | 13.58.97.152 | 44,864,412 | ~42MB |
| admin | 143.110.233.10 | 31,009,387 | ~29MB |
| svc_upload | 13.58.97.152 | 15,101,314 | ~14MB |

### Key Observations
- `backupsvc` is the most active exfiltration account 
  sending 533MB to a single external IP — likely a 
  backup service account abused for large data theft
- High privilege accounts `admin` and `system` both 
  compromised — attacker had significant access
- Data sent to 3 different external IPs suggesting 
  attacker distributed exfiltration to avoid detection
- FTP (port 21) is unencrypted — attacker made no 
  effort to hide the content of transfers
- `8.8.8.8` (Google DNS) receiving FTP transfers is 
  highly anomalous — possible DNS tunneling or 
  spoofed destination

### Suspicious External IPs
| IP | Reason Flagged |
|----|---------------|
| 89.45.23.77 | Unknown external — received 533MB |
| 143.110.233.10 | Unknown external — multiple accounts |
| 13.58.97.152 | Unknown external — multiple accounts |
| 8.8.8.8 | Google DNS — should not receive FTP traffic |

## Severity
**Critical** — Confirmed large scale data exfiltration.
Multiple high privilege accounts compromised. Over 1GB
of data sent to external destinations.

## MITRE ATT&CK Mapping
| Tactic | Technique ID | Technique Name |
|--------|-------------|----------------|
| Exfiltration | T1048 | Exfiltration Over Alternative Protocol |
| Exfiltration | T1048.003 | Exfiltration Over Unencrypted Protocol |
| Command & Control | T1071.002 | File Transfer Protocols |
| Initial Access | T1078 | Valid Accounts |

## Recommended Response
1. Immediately disable accounts: `backupsvc`, 
   `svc_upload`, `admin`, `system` pending investigation
2. Block all 4 external IPs at the firewall immediately
3. Preserve all FTP logs for forensic investigation
4. Determine what data was exfiltrated and assess 
   regulatory breach notification requirements
5. Investigate how attacker gained access to 4 
   separate privileged accounts
6. Audit all other service accounts for similar activity
7. Disable FTP entirely — replace with SFTP which 
   encrypts transfers
8. Implement DLP (Data Loss Prevention) controls to 
   alert on large outbound transfers

## Screenshots
[Screenshot 1 — FTP traffic on port 21]
<img width="2880" height="1539" alt="Screenshot 2026-07-09 031630" src="https://github.com/user-attachments/assets/69009ff7-66e8-489f-b138-6589608651d0" />

[Screenshot 2 — Large transfers filtered by bytes_out]
<img width="2880" height="1539" alt="Screenshot 2026-07-09 031817" src="https://github.com/user-attachments/assets/c5007c0e-6a91-453a-a7c7-7e107547cd8c" />

[Screenshot 3 — svc_upload timeline with dest IPs]
<img width="2880" height="1532" alt="Screenshot 2026-07-09 032107" src="https://github.com/user-attachments/assets/91c68469-1078-4497-ab9b-69dfd8ad08f1" />

[Screenshot 4 — Total bytes by user and destination]
<img width="2880" height="1530" alt="Screenshot 2026-07-09 033547" src="https://github.com/user-attachments/assets/dc988e3a-8f0e-40fe-9b65-f3db4078ff28" />

[Screenshot 5 — Total bytes exfiltrated by svc_upload]
<img width="2880" height="1512" alt="Screenshot 2026-07-09 032802" src="https://github.com/user-attachments/assets/75b2b36c-da5c-4661-a3e1-fb7b7587981d" />

