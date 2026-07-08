
DNS Beaconing Detection
Objective

Investigate DNS traffic to identify hosts exhibiting DNS beaconing behavior, determine the affected system, identify the suspicious domain being contacted, and assess potential Command-and-Control (C2) communication.

Environment

SIEM: Splunk Enterprise 10.4

Log Source: Suricata DNS Logs (DNS_logs.json)

Host: LinuxUbuntuServer1

Index: suricata

Sourcetype: dns:suricata

SPL Queries Used

DNS query frequency by source host

index="suricata" sourcetype="dns:suricata"
| stats count by src_ip, dns.rrname

Identify hosts with excessive DNS queries

index="suricata" sourcetype="dns:suricata"
| stats count by src_ip, dns.rrname
| where count > 50

Review DNS beacon timeline

index="suricata" sourcetype="dns:suricata" src_ip="192.168.1.101"
| table timestamp dns.rrname
| sort timestamp

Display victim host, queried domain, and destination DNS server

index="suricata" sourcetype="dns:suricata" src_ip="192.168.1.101"
| table timestamp src_ip dns.rrname dest_ip
Findings
Overview
100 DNS events analyzed.
10 internal hosts generated DNS traffic.
Most hosts queried legitimate services such as Google, Microsoft, Cloudflare, and Dropbox.
One internal host generated an unusually high number of requests to a suspicious domain.
Suspicious Host
Source IP	Suspicious Domain	DNS Requests
192.168.1.101	b6f93jks.cmdcontrol-malware.com	76
Normal DNS Activity

Other internal hosts generated only a small number of DNS requests to well-known legitimate services including:

www.google.com
update.microsoft.com
cdn.cloudflare.com
api.dropbox.com

Request counts ranged from 1–3 queries, representing expected user activity.

Attack Pattern

Host 192.168.1.101 repeatedly queried the domain

b6f93jks.cmdcontrol-malware.com

throughout the investigation period.

The repeated, periodic requests to the same domain strongly indicate DNS beaconing, where malware continually contacts a Command-and-Control (C2) infrastructure to receive instructions or maintain connectivity.

Infrastructure Observed
Victim Host	Queried Domain	Destination IP
192.168.1.101	b6f93jks.cmdcontrol-malware.com	8.8.8.8

Important Note

The dest_ip shown in the DNS logs (8.8.8.8) is Google Public DNS, which acts as the DNS resolver receiving the query.

It is not the attacker's Command-and-Control server.

The malicious infrastructure identified during this investigation is the suspicious domain:

b6f93jks.cmdcontrol-malware.com

The actual attacker-controlled IP address would require DNS response records, passive DNS data, firewall logs, or additional network telemetry to identify.

Severity

High — A single internal host demonstrated clear DNS beaconing behavior by repeatedly querying a suspicious domain associated with potential Command-and-Control communication.

MITRE ATT&CK Mapping
Tactic	Technique ID	Technique Name
Command and Control	T1071.004	Application Layer Protocol: DNS

T1071.004 — Malware communicates with its Command-and-Control infrastructure using DNS requests to evade detection while maintaining remote connectivity.

Recommended Response
Immediately isolate 192.168.1.101 from the network.
Block the malicious domain b6f93jks.cmdcontrol-malware.com using DNS filtering or protective DNS services.
If the domain resolves to one or more IP addresses during further investigation, block those attacker IP addresses at the firewall and perimeter security devices.
Add the domain and any resolved IP addresses to IDS/IPS detection and blocking rules.
Perform a full malware scan and forensic investigation on the affected endpoint.
Review firewall, proxy, and endpoint logs for additional communication with the same domain or any IP addresses associated with it.
Hunt across the environment for other systems querying b6f93jks.cmdcontrol-malware.com.
Reset credentials and monitor the affected system for continued beaconing or persistence mechanisms after remediation.
Screenshots
DNS query frequency by source IP and domain
High-frequency DNS beacon detection (count > 50)
Beaconing timeline for 192.168.1.101
Victim host, queried domain, and destination DNS resolver table

Screesnshots
<img width="2880" height="1532" alt="Screenshot 2026-07-08 211722" src="https://github.com/user-attachments/assets/07d2d0ea-6d73-4cc0-91b1-f0a9033c0e4c" />
<img width="2880" height="1527" alt="Screenshot 2026-07-08 211808 - Copy" src="https://github.com/user-attachments/assets/34cd6211-5d2d-4624-b671-d39888ba53c2" />
<img width="2880" height="1527" alt="Screenshot 2026-07-08 211808 - Copy" src="https://github.com/user-attachments/assets/b5869bbc-96ab-4fba-b395-8a178696641c" />
<img width="2880" height="1527" alt="Screenshot 2026-07-08 211808 - Copy - Copy" src="https://github.com/user-attachments/assets/e691a4e2-109d-4bfe-9b2e-8373a1112962" />


