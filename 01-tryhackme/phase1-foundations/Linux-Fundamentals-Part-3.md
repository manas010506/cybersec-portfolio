# Linux Fundamentals Part 3

## Objective

Learn users, permissions, SSH, processes, and networking basics.

---

## Users and Permissions

### Permission Structure

```text
rwxrwxrwx
```

Represents:

```text
Owner
Group
Others
```

### Permission Values

| Value | Meaning |
|---------|----------|
| 4 | Read |
| 2 | Write |
| 1 | Execute |

---

## chmod

### Example

```bash
chmod 755 script.sh
```

Permissions:

```text
Owner = rwx
Group = r-x
Others = r-x
```

### Example

```bash
chmod 644 file.txt
```

Permissions:

```text
Owner = rw-
Group = r--
Others = r--
```

---

## sudo

```bash
sudo command
```

Runs a command with elevated privileges.

---

## SSH

### Connect to a Remote System

```bash
ssh user@ip
```

Used for remote administration.

---

## Process Management

### List Processes

```bash
ps aux
```

### Monitor Processes

```bash
top
```

### Kill a Process

```bash
kill PID
```

---

## Networking Commands

### View Network Interfaces

```bash
ip a
```

### Test Connectivity

```bash
ping google.com
```

### Check Listening Ports

```bash
ss -tulnp
```

---

## Important Files

### User Information

```text
/etc/passwd
```

### Password Hashes

```text
/etc/shadow
```

---

## Key Concepts

- Permissions control access to files.
- SSH enables secure remote access.
- sudo provides temporary administrative access.
- Processes can be monitored and terminated.
- Networking commands assist with troubleshooting.

---

## Commands to Remember

```bash
chmod
sudo
ssh
ps
top
kill
ip a
ping
ss
```

---

## Interview Questions

### Difference between 644 and 755?

644:
- Owner can read/write.
- Others can only read.

755:
- Owner can read/write/execute.
- Others can read and execute.

### Difference between /etc/passwd and /etc/shadow?

- passwd stores user account information.
- shadow stores password hashes.

### What is SSH?

A secure protocol used for remote administration.

### Why use sudo?

To perform administrative actions without logging in as root.

---

## Cybersecurity Relevance

- SSH is heavily used in AWS EC2 management.
- Permission misconfigurations often lead to privilege escalation.
- Process analysis is important during incident investigations.
- Linux administration skills are required for Cloud Security and SOC roles.

---

## Skills Acquired

- User management
- Permission management
- SSH administration
- Process monitoring
- Network troubleshooting