# DNS Beaconing Detection

## Objective

Investigate DNS traffic to identify hosts exhibiting DNS beaconing behavior, determine the affected system, identify the suspicious domain being contacted, and assess potential Command-and-Control (C2) communication.

---

# Environment

- **SIEM:** Splunk Enterprise 10.4
- **Log Source:** Suricata DNS Logs (`DNS_logs.json`)
- **Host:** LinuxUbuntuServer1
- **Index:** suricata
- **Sourcetype:** dns:suricata

---

# SPL Queries Used

## DNS query frequency by source host

```spl
index="suricata" sourcetype="dns:suricata"
| stats count by src_ip, dns.rrname
```

## Identify hosts with excessive DNS queries

```spl
index="suricata" sourcetype="dns:suricata"
| stats count by src_ip, dns.rrname
| where count > 50
```

## DNS beaconing timeline

```spl
index="suricata" sourcetype="dns:suricata" src_ip="192.168.1.101"
| table timestamp dns.rrname
| sort timestamp
```

## Victim to destination IP mapping

```spl
index="suricata" sourcetype="dns:suricata" src_ip="192.168.1.101"
| table timestamp src_ip dns.rrname dest_ip
```

---

# Findings

## Overview

Analysis of the Suricata DNS logs identified one internal host generating an unusually high volume of repetitive DNS requests.

Out of **100 DNS events**, **76 queries** originated from a single system:

- **Victim Host:** `192.168.1.101`
- **Suspicious Domain:** `b6f93jks.cmdcontrol-malware.com`
- **Query Count:** **76**
- **Destination DNS Server:** `8.8.8.8`

All remaining hosts generated only one to three DNS queries to legitimate services such as Google, Microsoft, Dropbox, and Cloudflare.

---

## DNS Query Frequency

| Source IP | Domain | Query Count |
|-----------|--------|------------:|
| **192.168.1.101** | **b6f93jks.cmdcontrol-malware.com** | **76** |

---

## Timeline Analysis

The timestamp investigation shows the infected host repeatedly querying the same malicious domain over short intervals.

Example events:

| Timestamp | Domain |
|-----------|--------|
| 2025-08-02T07:00:00+00:00 | b6f93jks.cmdcontrol-malware.com |
| 2025-08-02T07:00:43+00:00 | b6f93jks.cmdcontrol-malware.com |
| 2025-08-02T07:01:00+00:00 | b6f93jks.cmdcontrol-malware.com |
| 2025-08-02T07:01:39+00:00 | b6f93jks.cmdcontrol-malware.com |
| 2025-08-02T07:04:40+00:00 | b6f93jks.cmdcontrol-malware.com |
| ... | ... |

The repeated queries at consistent intervals are characteristic of **DNS beaconing**, where malware periodically checks in with its Command-and-Control infrastructure.

---

## Victim-to-Destination Mapping

| Victim (Source IP) | Suspicious Domain | Destination IP |
|-------------------|-------------------|----------------|
| 192.168.1.101 | b6f93jks.cmdcontrol-malware.com | 8.8.8.8 |

The `dest_ip` observed in these DNS events is **8.8.8.8**, Google's public DNS resolver.

**Important:** This is **not** the attacker's web server.

Instead:

```
Victim
192.168.1.101
      │
      ▼
Google DNS (8.8.8.8)
      │
      ▼
b6f93jks.cmdcontrol-malware.com
      │
      ▼
Attacker Infrastructure
```

The actual attacker IP cannot be determined from the DNS logs alone because the victim is communicating with the DNS resolver, which then resolves the malicious domain.

To identify the attacker’s actual infrastructure, additional HTTP, TLS, or network flow logs would be required after DNS resolution.

---

# Attack Pattern

The infected workstation repeatedly queried the same suspicious domain while all other hosts generated only normal DNS activity.

This behavior is highly indicative of malware establishing periodic communication with its Command-and-Control server using DNS beaconing.

---

# Severity

**High**

A single host demonstrates repeated communication with a suspicious domain commonly associated with malware Command-and-Control activity.

---

# MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique |
|---------|--------------|-----------|
| Command and Control | T1071.004 | Application Layer Protocol: DNS |
| Command and Control | T1008 | Fallback Channels |

**T1071.004** – Malware uses DNS as its communication channel with C2 infrastructure.

**T1008** – DNS provides an alternate communication channel that is often allowed through firewalls.

---

# Recommended Response

- Immediately isolate host **192.168.1.101** from the network.
- Block access to **b6f93jks.cmdcontrol-malware.com** at the DNS filtering solution or firewall.
- If the domain has known resolved IP addresses, block those IPs at the firewall.
- Perform malware and endpoint forensic analysis on the affected workstation.
- Review HTTP, HTTPS, and proxy logs to identify any follow-up communication after DNS resolution.
- Monitor the environment for additional hosts querying the same domain.
- Add the domain to DNS sinkhole or threat intelligence blocklists.
- Reset credentials if compromise of the affected system is suspected.

---

# Screenshots

- DNS query frequency by source host
  <img width="2880" height="1532" alt="Screenshot 2026-07-08 211722" src="https://github.com/user-attachments/assets/90f788c0-5ea8-4874-8671-70463eb8b912" />

- Excessive DNS query detection
  <img width="2880" height="1532" alt="Screenshot 2026-07-08 211722" src="https://github.com/user-attachments/assets/b28b8af2-d8e3-40f4-85f7-c222517b9c08" />

- DNS beaconing timeline
  <img width="2880" height="1536" alt="Screenshot 2026-07-08 212440" src="https://github.com/user-attachments/assets/3ae7292e-f7f6-412c-a413-b71e2b7a2455" />

- Victim-to-destination IP mapping
  <img width="2880" height="1536" alt="Screenshot 2026-07-08 212440" src="https://github.com/user-attachments/assets/0edbe772-dfa1-4326-9be9-07b1a2e2a38c" />
