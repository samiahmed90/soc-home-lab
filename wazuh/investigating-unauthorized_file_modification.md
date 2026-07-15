<img width="2880" height="1704" alt="image" src="https://github.com/user-attachments/assets/00bec23f-7741-4082-b898-5b2b9d0401e5" /># Investigating Unauthorized File Modification Using Wazuh

## Overview

This project demonstrates how to configure **Wazuh File Integrity Monitoring (FIM)** to detect unauthorized changes on an Ubuntu Linux system. The Syscheck module was configured to monitor the `/root` directory in real time. After applying the configuration, file creation, modification, and deletion events were generated to validate the monitoring process.

---

## Lab Environment

| Component | Value |
|-----------|-------|
| SIEM | Wazuh |
| Operating System | Ubuntu Linux |
| Monitoring Module | Syscheck (File Integrity Monitoring) |
| Configuration File | `/var/ossec/etc/ossec.conf` |
| Monitored Directory | `/root` |

---

## Configuring File Integrity Monitoring

Navigate to the Wazuh configuration file.

```bash
cd /var/ossec/etc
nano ossec.conf
```

Inside the `<syscheck>` block, add the following configuration.

```xml
<directories check_all="yes" report_changes="yes" realtime="yes">/root</directories>
```

Restart the Wazuh agent to apply the changes.

```bash
sudo systemctl restart wazuh-agent
```

---

## Validation

### Step 1 - Create a File

Create a file inside the monitored directory.

```bash
nano /root/FIM.txt
```

Expected Result

- Rule ID **554**
- Event: **File Added**

---

### Step 2 - Modify the File

Edit the file by adding additional content.

Expected Result

- Rule ID **550**
- Event: **File Modified**

---

### Step 3 - Delete the File

Delete the monitored file.

```bash
rm /root/FIM.txt
```

Expected Result

- Rule ID **553**
- Event: **File Deleted**

---

## Detection Results

The Wazuh dashboard successfully detected every file operation performed during testing.

| Rule ID | Event | File |
|---------|-------|------|
| 554 | File Added | `/root/FIM.txt` |
| 550 | File Modified | `/root/test.txt` |
| 553 | File Deleted | `/root/FIM.txt` |

### Wazuh Dashboard

![File Integrity Monitoring Results](images/fim-events.png)

The dashboard confirms:

- File creation detected
- File modification detected
- File deletion detected
- Event timestamps
- Rule IDs
- File paths
- Agent responsible for the event

---

## How File Integrity Monitoring Works

```text
File Operation
      │
      ▼
Linux inotify
      │
      ▼
Wazuh Agent (Syscheck)
      │
      ▼
File hash & metadata comparison
      │
      ▼
Wazuh Manager
      │
      ▼
Dashboard Alert
```

---

## Skills Demonstrated

- Wazuh SIEM
- File Integrity Monitoring (FIM)
- Syscheck Configuration
- Linux Administration
- Host-Based Intrusion Detection (HIDS)
- Security Monitoring
- Security Event Analysis
- Incident Detection

---

## Conclusion

This project successfully demonstrated the deployment and validation of Wazuh File Integrity Monitoring on Ubuntu Linux. By monitoring the `/root` directory in real time, Wazuh accurately detected file creation, modification, and deletion events, demonstrating how FIM can be used to identify unauthorized changes and improve host-based threat detection.

---

## Screenshots

### 1. Wazuh File Integrity Monitoring Configuration

The `/root` directory was added to the `<syscheck>` section of `ossec.conf` to enable real-time File Integrity Monitoring.

<img width="2178" height="1226" alt="Screenshot 2026-07-15 223624" src="https://github.com/user-attachments/assets/bd7728c8-f2ec-4672-8051-dfcd404810a9" />


---

### 2. Restarting the Wazuh Agent

After updating the configuration, the Wazuh agent was restarted to apply the changes.

<img width="2178" height="1226" alt="Screenshot 2026-07-15 223842" src="https://github.com/user-attachments/assets/e80b7b0a-0a9a-4d2b-a9aa-87c17aa33365" />


---

### 3. Creating the Test File

A test file (`FIM.txt`) was created inside the monitored `/root` directory to generate a file creation event.

<img width="2178" height="1226" alt="Screenshot 2026-07-15 224509" src="https://github.com/user-attachments/assets/21b1252c-1366-46d3-bebd-67d17c967217" />


---

### 4. Modifying the Test File

The contents of the monitored file were modified to verify that Wazuh detects integrity changes.

<img width="2350" height="1226" alt="Screenshot 2026-07-15 225501" src="https://github.com/user-attachments/assets/c48d3362-4cec-4b62-ba60-111e447820e7" />


---

### 5. Deleting the Test File

The monitored file was deleted to generate a file deletion event.

<img width="2350" height="1226" alt="image" src="https://github.com/user-attachments/assets/0f003ca5-4427-4cb7-9abb-d0388ce9ddbb" />


---

### 6. Wazuh Detection Results

The Wazuh Dashboard successfully detected all file integrity events, including file creation, modification, and deletion. The generated alerts include the monitored file path, event type, rule description, severity level, and corresponding rule ID.

<img width="2880" height="1511" alt="Screenshot 2026-07-15 231956" src="https://github.com/user-attachments/assets/2bec3f18-59c9-4e75-bde8-e9462142857b" />

