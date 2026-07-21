# Osquery Investigation Report: Listing Installed Software on Linux

## Objective
The objective of this investigation was to use **osquery** to enumerate installed software on an Ubuntu Linux system, identify a specific package (Nmap), verify its installation details, examine its file metadata, and confirm that the osquery daemon was actively running.

---

# Environment

| Component | Details |
|----------|---------|
| Operating System | Ubuntu 24.04 LTS |
| Platform | Oracle VirtualBox |
| Tool | osquery |
| Shell | osquery Interactive Shell (`osqueryi`) |

---

# Investigation Steps

## 1. Launch the Osquery Interactive Shell

The investigation began by launching the interactive osquery shell.

```sql
osqueryi
```

This opened the SQL-based interface, allowing system information to be queried using SQL syntax.

---

## 2. Enumerate Installed Software

To list every package installed on the Ubuntu system, the following query was executed:

```sql
SELECT name, version
FROM deb_packages;
```

### Result

The query successfully returned every installed Debian package along with its installed version.

Examples of packages observed include:

- accountservice
- acl
- apparmor
- apt
- xz-utils
- zip
- zstd

This demonstrates how osquery can quickly inventory all installed software on a Linux endpoint.

---

## 3. Search for Nmap Installation

Instead of manually searching through hundreds of packages, the package list was filtered for Nmap.

```sql
SELECT name, version
FROM deb_packages
WHERE name LIKE '%nmap%';
```

### Result

The query identified the following installed packages:

| Package | Version |
|---------|----------|
| nmap | 7.94+git20230807.3be01efb1+dfsg-3build2 |
| nmap-common | 7.94+git20230807.3be01efb1+dfsg-3build2 |

This confirmed that Nmap was installed along with its supporting package.

---

## 4. Verify Nmap File Metadata

After confirming installation, the executable's file metadata was inspected.

```sql
SELECT path,
       datetime(mtime, 'unixepoch') AS modified_time
FROM file
WHERE path LIKE '/usr/bin/nmap%'
   OR path LIKE '/usr/lib/%nmap';
```

### Result

The query returned the executable path and its last modification timestamp.

Example:

| Path | Modified Time |
|------|---------------|
| /usr/bin/nmap | 2024-04-01 06:58:48 |

This information can be useful during forensic investigations to determine when software was last modified.

---

## 5. Verify the Osquery Daemon

Finally, the running osquery process was verified.

```sql
SELECT pid,
       name,
       path
FROM processes
WHERE path LIKE '%osquery%';
```

### Result

The query confirmed that the osquery daemon was running.

| PID | Process | Path |
|-----|----------|--------------------------|
| 4234 | osqueryi | /opt/osquery/bin/osqueryd |

This confirms that the osquery service was active and functioning properly on the endpoint.

---

# Findings

- Successfully enumerated all installed software on the Ubuntu system.
- Confirmed the installation of **Nmap** and identified its installed version.
- Verified the file path and last modification timestamp of the Nmap executable.
- Confirmed that the **osquery daemon (`osqueryd`)** was actively running.
- Demonstrated that osquery provides comprehensive endpoint visibility through SQL-based queries.

---

# Conclusion

This investigation demonstrated how **osquery** can be used as an endpoint visibility and forensic tool to gather detailed information about installed software on a Linux system. Using SQL queries, it was possible to enumerate installed packages, locate specific software, verify executable metadata, and confirm active system processes. These capabilities make osquery a valuable tool for system administrators, security analysts, and incident responders when auditing endpoints or performing forensic investigations.

---

# Screenshots

### Figure 1. Listing all installed software using the `deb_packages` table.

<img width="1316" height="1280" alt="Screenshot 2026-07-21 221917" src="https://github.com/user-attachments/assets/dadbb7cf-5d4a-4c87-b1a1-9e72c7c3416c" />


---

### Figure 2. Filtering installed packages to locate Nmap.

<img width="1316" height="1280" alt="Screenshot 2026-07-21 222013" src="https://github.com/user-attachments/assets/590b7a42-1fee-40b9-a839-2c5b5623138c" />


---

### Figure 3. Verifying the Nmap executable path and last modification timestamp.

<img width="1162" height="660" alt="Screenshot 2026-07-21 223403" src="https://github.com/user-attachments/assets/ebf15cef-2529-44e8-84f7-b676c2a83a7e" />


---

### Figure 4. Confirming the osquery daemon is actively running.
<img width="1316" height="1280" alt="Screenshot 2026-07-21 223902" src="https://github.com/user-attachments/assets/0212e879-8c9f-459a-b266-cbbd3300c50b" />



