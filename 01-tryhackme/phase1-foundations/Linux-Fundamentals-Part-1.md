# Linux Fundamentals Part 1

## Objective
Learn Linux basics, filesystem navigation, and essential Linux commands.

---

## Linux Directory Structure

| Directory | Purpose |
|------------|----------|
| / | Root directory |
| /home | User home directories |
| /root | Root user home directory |
| /etc | Configuration files |
| /var | Logs and variable data |
| /tmp | Temporary files |
| /bin | Essential binaries |
| /usr | User programs and utilities |

---

## Commands Learned

### Print Working Directory

```bash
pwd
```

Displays the current directory.

### List Files

```bash
ls
ls -la
```

Lists files and directories.

### Change Directory

```bash
cd
cd ..
cd ~
```

Navigate between directories.

### Create Directory

```bash
mkdir test
```

Creates a new directory.

### Remove Directory

```bash
rmdir test
```

Removes an empty directory.

### Create File

```bash
touch file.txt
```

Creates a new file.

---

## Key Concepts

- Linux is case-sensitive.
- Everything in Linux is treated as a file.
- The root directory is represented by `/`.
- Hidden files begin with a dot (`.`).

---

## Commands to Remember

```bash
pwd
ls
ls -la
cd
mkdir
rmdir
touch
```

---

## Interview Questions

### What is Linux?

Linux is an open-source operating system widely used on servers, cloud platforms, and security appliances.

### What does pwd do?

Displays the current working directory.

### What is the root directory?

The top-most directory in Linux represented by `/`.

### Why are hidden files important?

They often contain configuration information.

---

## Cybersecurity Relevance

- Most cloud servers run Linux.
- Security analysts often investigate Linux systems.
- Linux knowledge is required for SOC, Cloud Security, and Penetration Testing roles.

---

## Skills Acquired

- Linux navigation
- Directory management
- File creation
- Command-line usage