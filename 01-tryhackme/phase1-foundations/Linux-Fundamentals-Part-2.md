# Linux Fundamentals Part 2

## Objective

Learn file operations and text handling in Linux.

---

## File Operations

### Copy Files

```bash
cp file.txt backup.txt
```

Copies a file.

### Move or Rename Files

```bash
mv file.txt newfile.txt
```

Moves or renames a file.

### Delete Files

```bash
rm file.txt
```

Deletes a file.

### Delete Directories

```bash
rm -r folder
```

Deletes a directory recursively.

---

## Viewing File Contents

### Display Entire File

```bash
cat file.txt
```

### View First Lines

```bash
head file.txt
```

### View Last Lines

```bash
tail file.txt
```

### Read Large Files

```bash
less file.txt
```

---

## Searching

### Search Text

```bash
grep "error" logfile.log
```

Searches for matching text.

### Find Files

```bash
find / -name file.txt
```

Searches for files.

---

## Key Concepts

- cp copies files.
- mv moves or renames files.
- rm permanently deletes files.
- grep is commonly used for log analysis.
- find locates files within the filesystem.

---

## Commands to Remember

```bash
cp
mv
rm
cat
head
tail
less
grep
find
```

---

## Interview Questions

### Difference between cp and mv?

cp copies files while mv moves or renames files.

### What does grep do?

Searches for patterns within text files.

### Why use tail?

To inspect the latest log entries.

### Why use find?

To locate files on a system.

---

## Cybersecurity Relevance

- Security analysts frequently search logs using grep.
- Log analysis often involves cat, less, head, and tail.
- Finding suspicious files is a common investigation task.

---

## Skills Acquired

- File management
- Log inspection
- Text searching
- File discovery