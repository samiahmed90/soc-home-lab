# Windows Account Investigation

## Objective

Investigate suspicious Windows authentication activity for the **ADMIN_JAIMIN** account to determine whether the account shows signs of compromise by analyzing authentication events, source IP addresses, login patterns, and privileged access.

---

## Environment

| Item | Value |
|------|-------|
| SIEM | Splunk Enterprise 10.4 |
| Log Source | Windows Security Logs (`windows_admin_login_logs_full.json`) |
| Host | LinuxUbuntuServer1 |
| Index | `main` |
| Sourcetype | `windows_security` |

---

# SPL Queries Used

### View successful logons

```spl
index="main" sourcetype="windows_security" Account_Name="ADMIN_JAIMIN" EventCode=4624
| table timestamp src_ip Logon_Type
| sort timestamp
```

### Identify source IPs

```spl
index="main" sourcetype="windows_security" Account_Name="ADMIN_JAIMIN" EventCode=4624
| stats count by src_ip
| sort -count
```

### Check failed logons

```spl
index="main" sourcetype="windows_security" Account_Name="ADMIN_JAIMIN" EventCode=4625
| stats count by src_ip
| sort -count
```

### Check privileged logons

```spl
index="main" sourcetype="windows_security" Account_Name="ADMIN_JAIMIN" EventCode=4672
| table timestamp src_ip
```

---

# Findings

## Overview

- **29** successful logons (Event ID **4624**)
- **2** failed logon attempts (Event ID **4625**)
- **1** privileged logon event (Event ID **4672**)
- Authentication originated from **6 different external IP addresses**

---

## Successful Logons by Source IP

| Source IP | Successful Logons |
|-----------|------------------:|
| 106.51.24.89 | 7 |
| 103.21.244.0 | 6 |
| 185.220.101.1 | 6 |
| 88.198.59.200 | 5 |
| 141.98.128.23 | 4 |
| 122.179.152.12 | 1 |

---

## Failed Logons

| Source IP | Failed Attempts |
|-----------|----------------:|
| 106.51.24.89 | 1 |
| 185.220.101.1 | 1 |

Only two failed authentication attempts were observed, indicating that most authentication requests were accepted successfully.

---

## Authentication Timeline

Partial timeline of successful RDP logons:

| Timestamp (UTC) | Source IP | Logon Type |
|-----------------|-----------|-----------:|
| 2025-08-02 00:00 | 185.220.101.1 | 10 |
| 2025-08-02 00:20 | 185.220.101.1 | 10 |
| 2025-08-02 00:35 | 185.220.101.1 | 10 |
| 2025-08-02 00:50 | 103.21.244.0 | 10 |
| 2025-08-02 00:55 | 141.98.128.23 | 10 |
| 2025-08-02 01:40 | 88.198.59.200 | 10 |
| 2025-08-02 01:55 | 88.198.59.200 | 10 |
| 2025-08-02 02:20 | 122.179.152.12 | 10 |

The timeline shows successful **Remote Desktop (Logon Type 10)** logons occurring at regular intervals while rotating between multiple external IP addresses. This behavior is inconsistent with typical administrator activity and suggests an automated process or repeated remote access using valid credentials.

---

## Privileged Access

One privileged logon event (Event ID **4672**) was identified.

| Timestamp | Source IP |
|-----------|-----------|
| 2025-08-02 10:10 UTC | 122.179.152.12 |

This confirms that administrative privileges were assigned following a successful authentication.

---

# Analysis

The investigation identified several suspicious indicators:

- Multiple successful Remote Desktop logons from **six different public IP addresses**
- Authentication events occurring at **consistent time intervals**
- Very few failed logon attempts
- Successful administrative logon (Event ID 4672)

These indicators suggest the account credentials were already valid and were being used repeatedly to establish Remote Desktop sessions from rotating external IP addresses.

Although the available logs cannot confirm a compromise, the activity warrants immediate investigation due to the unusual authentication pattern.

---

# Severity

**High**

The combination of repeated RDP logons, multiple external IP addresses, and privileged account access represents a high-risk authentication anomaly.

---

# MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique |
|--------|--------------|-----------|
| Initial Access | T1078 | Valid Accounts |
| Lateral Movement | T1021.001 | Remote Services: Remote Desktop Protocol |

**T1078 – Valid Accounts**

The account successfully authenticated multiple times, indicating valid credentials were used.

**T1021.001 – Remote Desktop Protocol**

Logon Type **10** confirms the account accessed the system through Remote Desktop Services.

---

# Recommended Response

- Verify whether all six source IP addresses belong to authorized administrators.
- Reset the password for the **ADMIN_JAIMIN** account.
- Review activity performed after each successful logon.
- Correlate authentication events with firewall, VPN, and endpoint telemetry.
- Enable Multi-Factor Authentication (MFA) for privileged accounts.
- Restrict RDP access using VPN or IP allowlists.
- Continue monitoring for additional logons from unfamiliar IP addresses.

---

# Screenshots

- Successful logons by source IP
  <img width="2880" height="1529" alt="Screenshot 2026-07-09 225348" src="https://github.com/user-attachments/assets/42a57096-c403-4a65-b502-969495e9e290" />
- Authentication timeline
  <img width="2858" height="1517" alt="Screenshot 2026-07-09 225616" src="https://github.com/user-attachments/assets/0549c30f-41d0-47a3-9f82-18386d644e36" />

- Failed logons (Event ID 4625)
  <img width="2880" height="1522" alt="Screenshot 2026-07-09 230332" src="https://github.com/user-attachments/assets/14c6c9e3-45ef-473c-a77c-92a93a3f981c" />

- Privileged logon (Event ID 4672)
  <img width="2880" height="1517" alt="Screenshot 2026-07-09 230411" src="https://github.com/user-attachments/assets/1810f226-0cbc-4a35-b208-2c7002bc33be" />

