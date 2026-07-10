# AWS Console Login Geolocation Investigation

## Objective

Investigate AWS CloudTrail **ConsoleLogin** events to identify successful AWS Management Console logins originating from locations outside the organization's expected login region (India). The objective is to determine whether the observed authentication activity may indicate unauthorized access, compromised credentials, or other suspicious behavior.

---

## Environment

| Component | Value |
| :-------- | :---- |
| **SIEM** | Splunk Enterprise 10.4 |
| **Log Source** | AWS CloudTrail |
| **Index** | `aws_logs` |
| **Sourcetype** | `aws:cloudtrail` |

---

## SPL Queries Used

### Display Console Login Events with Geolocation

```spl
index="aws_logs" sourcetype="aws:cloudtrail" eventName="ConsoleLogin"
| iplocation sourceIPAddress
| table _time userIdentity.userName sourceIPAddress Country
```

### Identify Successful Console Logins Outside India

```spl
index="aws_logs" sourcetype="aws:cloudtrail" eventName="ConsoleLogin" responseElements.ConsoleLogin="Success"
| iplocation sourceIPAddress
| where Country!="India"
| table _time userIdentity.userName sourceIPAddress Country responseElements.ConsoleLogin
```

---

## Investigation

The investigation began by reviewing AWS CloudTrail **ConsoleLogin** events to identify where users were authenticating to the AWS Management Console.

To enrich the authentication events with geographic information, the `iplocation` command was used to map each source IP address to its corresponding country. This provided visibility into the geographic origin of every successful login.

The initial query displayed all console login events, including the username, source IP address, and associated country. Review of the results showed that while several users authenticated from **India**, multiple successful logins also originated from **Brazil** and **Argentina**.

To focus on potentially suspicious activity, the investigation was refined to display only successful logins originating from countries other than India. The filtered results confirmed successful AWS Management Console logins from unexpected geographic locations involving multiple user accounts.

Because the organization expects AWS console access to originate only from India, authentication from foreign countries represents an authentication anomaly requiring further investigation.

---

## Findings

| Observation | Result |
| :---------- | :----- |
| Expected login location | India |
| Unexpected login locations | Brazil, Argentina |
| Successful foreign logins detected | Yes |
| Multiple user accounts affected | Yes |
| Administrative account involved | Yes (`admin_svc`) |

The investigation identified multiple successful AWS Management Console logins originating from **Brazil** and **Argentina**, despite **India** being the organization's expected login location.

The affected accounts included:

- `admin_svc`
- `dev_user1`
- `dev_user2`

Unlike failed authentication attempts, these events represent **successful logins**, meaning AWS accepted valid credentials for each session.

Although CloudTrail logs alone cannot determine whether the activity resulted from legitimate travel, approved VPN usage, or compromised credentials, the presence of successful authentication from unexpected countries should be treated as suspicious until validated.

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ATT&CK ID |
| :----- | :-------- | :-------- |
| Initial Access | Valid Accounts | **T1078** |


### **T1078 – Valid Accounts**

The successful AWS Console logins indicate that valid credentials were used to authenticate to the AWS Management Console. If the observed activity is unauthorized, this aligns with the use of legitimate credentials to gain access without exploiting software vulnerabilities.

---

## Recommended Response

- Validate whether affected users were traveling or using approved VPN services during the login times.
- Review CloudTrail activity immediately following each successful login for suspicious IAM, EC2, S3, or other administrative actions.
- Reset passwords and rotate access credentials for any accounts determined to be compromised.
- Enforce Multi-Factor Authentication (MFA) for all IAM users, especially privileged accounts.
- Configure SIEM alerts for successful AWS Console logins originating outside approved geographic regions.
- Investigate the source IP addresses using threat intelligence to determine whether they are associated with known malicious infrastructure.
- Review additional AWS CloudTrail logs for privilege escalation, persistence mechanisms, or unusual API activity following the detected logins.

---

## Screenshots

### Console Login Events with Geolocation

<img width="2880" height="1512" alt="Screenshot 2026-07-10 225928" src="https://github.com/user-attachments/assets/6c4d85d9-2e67-4b86-a125-b963b1422ba0" />


---

### Successful Console Logins Outside India

![Uploading Screenshot 2026-07-10 225928.png…]()


