# Lab 2: Detect New User Accounts Created with Osquery

## Objective

The objective of this lab was to use **Osquery** to enumerate local user accounts on a Linux system and detect when a new user account has been created. This type of investigation is useful for identifying unauthorized or suspicious accounts that may have been added by an attacker or insider.

---

## Environment

- **Operating System:** Ubuntu 24.04 LTS
- **Tool:** Osquery
- **Platform:** Oracle VirtualBox

---

## Step 1: Enumerate Existing User Accounts

I first queried the `users` table to list all local user accounts with a UID greater than or equal to **1000**, which generally represents regular human users on Linux systems.

### Query

```sql
SELECT username, uid, gid, description, directory, shell
FROM users
WHERE uid >= 1000
ORDER BY uid DESC;
```

### Initial Findings

The system contained two accounts:

| Username | UID | Description | Home Directory | Shell |
|----------|----:|-------------|----------------|-------|
| nobody | 65534 | nobody | /nonexistent | /usr/sbin/nologin |
| samiahmed90 | 1000 | samiahmed90 | /home/samiahmed90 | /bin/bash |

**Analysis**

- `samiahmed90` is the primary user account created during Ubuntu installation.
- The `nobody` account is a built-in system account used by Linux services and is not a regular login account.
- No additional user accounts were present during the initial investigation.

---

## Step 2: Check Logged-in Sessions

To determine whether the user account was currently active, I queried the `logged_in_users` table.

### Query

```sql
SELECT *
FROM logged_in_users
WHERE user = 'samiahmed90';
```

### Findings

Three active sessions were returned:

| User | Session | Host |
|------|---------|------|
| samiahmed90 | seat0 | login screen |
| samiahmed90 | tty2 | Local terminal |
| samiahmed90 | pts/1 | 192.168.68.110 |

### Analysis

The results indicate that the user currently has multiple active sessions:

- **seat0** represents the graphical login session.
- **tty2** represents a local virtual terminal.
- **pts/1** indicates a remote pseudo-terminal session connected from **192.168.68.110**.

This confirms the account is actively logged into the system.

---

## Step 3: Simulate Creation of a New User

To simulate a real security investigation, I created an additional user account.

### Command

```bash
sudo adduser testuser
```

During account creation:

- A new user named **testuser** was created.
- A new group with GID **1001** was automatically created.
- A home directory was created at:

```
/home/testuser
```

- The default Bash shell was assigned.
- User information was configured and the account creation completed successfully.

---

## Step 4: Verify the New User Account

After creating the account, I reran the original Osquery query.

### Query

```sql
SELECT username, uid, gid, description, directory, shell
FROM users
WHERE uid >= 1000
ORDER BY uid DESC;
```

### Findings

| Username | UID | Description | Home Directory | Shell |
|----------|----:|-------------|----------------|-------|
| nobody | 65534 | nobody | /nonexistent | /usr/sbin/nologin |
| testuser | 1001 | testuser90 | /home/testuser | /bin/bash |
| samiahmed90 | 1000 | samiahmed90 | /home/samiahmed90 | /bin/bash |

### Analysis

A newly created account named **testuser** appeared in the Osquery results.

Indicators confirming this account was newly created include:

- New username: **testuser**
- Newly assigned UID: **1001**
- Newly assigned GID: **1001**
- Home directory created at **/home/testuser**
- Default shell assigned as **/bin/bash**

This demonstrates how Osquery can quickly identify newly created local accounts during a security investigation.

---

# Conclusion

In this lab, Osquery was successfully used to enumerate local Linux user accounts and detect the addition of a newly created account.

By comparing the initial system state with the updated results after account creation, it was possible to identify the new user based on its username, UID, home directory, and assigned shell.

This technique is valuable during incident response because attackers often create persistence by adding unauthorized local user accounts. Regularly querying and comparing the `users` table allows security analysts to quickly identify unexpected account creation.

---

# Screenshots

### Screenshot 1 - Initial Enumeration of User Accounts

- Executed the `users` table query.
- Verified that only the default user account existed.
<img width="2350" height="1226" alt="Screenshot 2026-07-22 002116" src="https://github.com/user-attachments/assets/1dc571a9-2fb3-4b4a-b646-ffaeb6197ff9" />


---

### Screenshot 2 - Logged-in User Investigation

- Queried the `logged_in_users` table.
- Verified active sessions for the user `samiahmed90`.
<img width="2350" height="1226" alt="Screenshot 2026-07-22 002615" src="https://github.com/user-attachments/assets/795af8c8-aae4-4db9-ba80-397919c89c62" />


---

### Screenshot 3 - Creation of New User Account

- Created a new user named `testuser` using the `adduser` command.
- Ubuntu assigned a new UID, GID, home directory, and Bash shell.
<img width="2444" height="804" alt="Screenshot 2026-07-22 004421" src="https://github.com/user-attachments/assets/b4bcc0da-bd14-4d56-b5f8-9c4d1019ad8d" />


---

### Screenshot 4 - Verification of Newly Created Account

- Re-executed the Osquery query.
- Confirmed that `testuser` appeared in the results with UID **1001**.
- Verified the account's home directory and assigned shell.
<img width="2444" height="1336" alt="Screenshot 2026-07-22 004720" src="https://github.com/user-attachments/assets/93bec7b7-72a7-44c9-8b40-79671e6f0460" />

