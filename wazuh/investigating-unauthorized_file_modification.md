# Investigating Unauthorized File Modification Using Wazuh

## Overview

This project demonstrates how to configure **Wazuh File Integrity Monitoring (FIM)** to detect unauthorized file changes on an Ubuntu Linux system. The Wazuh **Syscheck** module was configured to monitor the `/root` directory in real time, allowing file creation, modification, and deletion events to be detected immediately.

---

## Objectives

- Configure Wazuh File Integrity Monitoring (FIM)
- Monitor the `/root` directory
- Detect file creation
- Detect file modification
- Detect file deletion
- Validate alerts within the Wazuh Dashboard

---

## Environment

| Component | Value |
|----------|-------|
| SIEM | Wazuh |
| Operating System | Ubuntu Linux |
| Detection Module | Syscheck |
| Configuration File | `/var/ossec/etc/ossec.conf` |
| Monitored Directory | `/root` |

---

## Configuration

The following line was added inside the `<syscheck>` section of `ossec.conf`.

```xml
<directories check_all="yes" report_changes="yes" realtime="yes">/root</directories>
```

### Configuration Parameters

| Parameter | Description |
|-----------|-------------|
| `check_all="yes"` | Monitors file attributes and hashes |
| `report_changes="yes"` | Reports file content modifications |
| `realtime="yes"` | Uses Linux inotify for real-time monitoring |
| `/root` | Directory being monitored |

After saving the configuration, the Wazuh agent was restarted.

```bash
sudo systemctl restart wazuh-agent
```

---

## Test 1 – File Creation

A new file named `FIM.txt` was created inside `/root`.

```bash
nano FIM.txt
```

Sample content:

```text
Hey There!
This is a file integrity monitoring test.
```

### Result

Wazuh detected the new file.

| Rule ID | Event |
|---------|-------|
| **554** | File Added |

> **Screenshot**

```
<img width="2880" height="1524" alt="Screenshot 2026-07-15 224628" src="https://github.com/user-attachments/assets/ac283afa-ecc6-47ab-8ec1-e73cb2c22c23" />

```

---

## Test 2 – File Modification

The file "test.txt" was modified by adding additional content.

```bash
nano test.txt
```

### Result

Wazuh detected the file integrity change.

| Rule ID | Event |
|---------|-------|
| **550** | Integrity Checksum Changed |

> **Screenshot**

```
<img width="2880" height="1503" alt="Screenshot 2026-07-15 225450" src="https://github.com/user-attachments/assets/4bf07169-31e0-41b8-ad68-754544c1ba03" />

```

---

## Test 3 – File Deletion

The monitored file was deleted.

```bash
rm FIM.txt
```

### Result

Wazuh detected the deletion.

| Rule ID | Event |
|---------|-------|
| **553** | File Deleted |

> **Screenshot**

```
<img width="2880" height="1503" alt="Screenshot 2026-07-15 225450" src="https://github.com/user-attachments/assets/ede9a929-290b-4243-95cd-2db619cadf9d" />
mages/file-deleted.png
```

---

## Wazuh Dashboard

The File Integrity Monitoring dashboard displayed alerts for all three events.

| Operation | Rule ID | Status |
|-----------|---------|--------|
| File Created | 554 | ✅ Detected |
| File Modified | 550 | ✅ Detected |
| File Deleted | 553 | ✅ Detected |

---

## What Happened Behind the Scenes

1. A file operation occurred inside the monitored directory.
2. Linux **inotify** detected the filesystem event.
3. The Wazuh agent processed the change.
4. Syscheck recalculated the file metadata and hash.
5. The event was sent to the Wazuh Manager.
6. The Wazuh Dashboard generated the corresponding alert.

---

## Detection Summary

| Rule ID | Description |
|---------|-------------|
| 550 | File Modified |
| 553 | File Deleted |
| 554 | File Added |

---

## Skills Demonstrated

- Wazuh
- File Integrity Monitoring (FIM)
- Syscheck Configuration
- Linux Administration
- Host-Based Intrusion Detection (HIDS)
- Security Monitoring
- Security Event Analysis
- Incident Detection

---

## Conclusion

This project successfully demonstrated Wazuh's File Integrity Monitoring capabilities by configuring the Syscheck module to monitor the `/root` directory in real time. Wazuh accurately detected file creation, modification, and deletion events, confirming that the monitoring configuration was functioning as expected. These capabilities are essential for identifying unauthorized changes to critical files and supporting incident detection and response on Linux systems.
