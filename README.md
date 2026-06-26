# SOC Home Lab

A hands-on home lab simulating real SOC analyst workflows 
including attack simulation, log analysis, and security 
investigation across multiple tools.

## Lab Architecture
- **Attacker:** Kali Linux (VirtualBox VM)
- **Victim/Server:** Ubuntu Server 26.04 LTS (VirtualBox VM)
- **Host:** Windows 11

## Tools
- **Splunk Enterprise 10.4** — SIEM, log analysis, dashboards
- **Wazuh** — EDR, threat detection, alert rules
- **Osquery** — Endpoint visibility and threat hunting

## Investigations

### Splunk
- SSH Brute Force Detection
- DNS Beaconing Detection
- Suspicious File Transfer Detection
- Compromised Windows User Account Investigation
- Unauthorized AWS Cloud Access Detection

### Wazuh
- Unauthorized File Modification Detection
- SSH Brute Force Investigation
- Vulnerability Detection
- SQL Injection Attack Detection

### Osquery
- List All Installed Software
- Detect New User Accounts Created
- Detect Malware Making Outbound Connections

## Skills Demonstrated
- SIEM deployment and configuration on Linux
- Attack simulation using Kali Linux
- Log ingestion and SPL query writing
- Endpoint detection and response
- Incident investigation and documentation
- Linux server administration via CLI and SSH

## Setup
See individual folders for setup guides and 
investigation writeups with screenshots.
