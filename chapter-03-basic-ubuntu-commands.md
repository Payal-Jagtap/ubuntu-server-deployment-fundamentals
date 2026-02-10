# Chapter 3 — Basic Ubuntu Commands

| | |
|---|---|
| **Objective** | Learn essential Linux commands for navigation, file management, and system usage |
| **Duration** | 30–40 minutes |
| **Difficulty** | Beginner |
| **Prerequisites** | Chapter 2 completed (Ubuntu installed and accessible) |

---

## 3.1 Overview

Now that you have a running Ubuntu system, it is time to learn how to navigate it through the **terminal**. While Ubuntu Desktop has a graphical interface, the terminal is where the real power lies — you type commands, and the system responds with text output. This is exactly how you will manage remote servers in production.

This chapter covers the fundamental commands you will use every day when working with Linux servers. Mastering these commands is essential before you can install software, configure services, or deploy applications.

---

## 3.2 Understanding the Concepts

### The Linux Directory Structure

Unlike Windows where files are organized under drive letters (`C:\`, `D:\`), Linux uses a single **tree structure** that starts from the root directory `/`.

```
/                          ← Root directory (the top of the tree)
├── home/                  ← Home directories for all users
│   └── ubuntu/            ← Your personal directory (~)
│       ├── Documents/
│       ├── Downloads/
│       └── practice/      ← Directories you create
├── etc/                   ← System configuration files
├── var/                   ← Variable data (logs, web files)
│   ├── log/               ← System log files
│   └── www/               ← Web server files (Nginx)
├── usr/                   ← User programs and utilities
│   ├── bin/               ← Command binaries (ls, cp, mv)
│   └── lib/               ← Shared libraries
├── tmp/                   ← Temporary files (cleared on reboot)
├── root/                  ← Home directory for the root (admin) user
├── bin/                   ← Essential command binaries
├── sbin/                  ← System administration binaries
├── opt/                   ← Optional/third-party software
└── dev/                   ← Device files
```

### Key Concepts

| Concept | Explanation |
|---------|-------------|
| **Path** | The address of a file or directory. Example: `/home/ubuntu/practice/file1.txt` |
| **Absolute path** | Starts from root `/`. Example: `/home/ubuntu/practice` |
| **Relative path** | Starts from your current location. Example: `practice/file1.txt` |
| **Home directory** | Your personal directory, represented by `~`. Usually `/home/yourusername` |
| **Current directory** | Where you are right now, represented by `.` |
| **Parent directory** | One level up from current, represented by `..` |

### The Command Prompt

When you log in, you see something like:

```
ubuntu@ubuntu-vm:~$
```

This tells you:
- `ubuntu` — your username
- `ubuntu-vm` — the computer's hostname
- `~` — your current location (home directory)
- `$` — you are a normal user (a `#` would mean you are the root/admin user)

---

## 3.3 Command Reference

### Navigation Commands

#### `pwd` — Print Working Directory

Shows your current location in the file system.

```bash
pwd
```

**Output:**
```
/home/ubuntu
```

**When to use:** When you are lost and need to know where you are.

---

#### `ls` — List Directory Contents

Shows files and folders in the current directory.

```bash
# Basic listing
ls

# Detailed listing (permissions, size, date)
ls -l

# Show hidden files (files starting with .)
ls -a

# Detailed listing including hidden files
ls -la

# List contents of a specific directory
ls /etc
```

**Example output of `ls -l`:**
```
drwxr-xr-x 2 ubuntu ubuntu 4096 Jan 15 10:30 practice
-rw-r--r-- 1 ubuntu ubuntu    0 Jan 15 10:31 file1.txt
```

**Reading the output:**
```
d rwx r-x r-x  2  ubuntu  ubuntu  4096  Jan 15 10:30  practice
│ │   │   │    │  │        │        │     │              │
│ │   │   │    │  │        │        │     │              └─ Name
│ │   │   │    │  │        │        │     └─ Last modified
│ │   │   │    │  │        │        └─ Size in bytes
│ │   │   │    │  │        └─ Group owner
│ │   │   │    │  └─ User owner
│ │   │   │    └─ Number of links
│ │   │   └─ Others' permissions
│ │   └─ Group permissions
│ └─ Owner permissions
└─ d=directory, -=file
```

---

#### `cd` — Change Directory

Move to a different directory.

```bash
# Go to a directory
cd practice

# Go up one level
cd ..

# Go to home directory
cd ~
# or simply
cd

# Go to an absolute path
cd /var/log

# Go to the previous directory
cd -
```

---

### File and Folder Management

#### `mkdir` — Make Directory

Create a new folder.

```bash
# Create a single directory
mkdir practice

# Create nested directories (parent and child at once)
mkdir -p projects/webapp/templates
```

---

#### `touch` — Create Empty File

Create a new empty file or update the timestamp of an existing file.

```bash
# Create a single file
touch file1.txt

# Create multiple files at once
touch file1.txt file2.txt file3.txt
```

---

#### `cp` — Copy Files and Directories

```bash
# Copy a file
cp file1.txt file1_backup.txt

# Copy a file to a directory
cp file1.txt practice/

# Copy an entire directory (use -r for recursive)
cp -r practice practice_backup
```

---

#### `mv` — Move or Rename Files

```bash
# Rename a file
mv file1.txt document.txt

# Move a file to a directory
mv document.txt practice/

# Move a directory
mv practice projects/
```

**Key difference between `cp` and `mv`:**
- `cp` = **copy** — the original file stays where it is, and a new copy is created
- `mv` = **move** — the original file is removed from its location and placed in the new location (also used for renaming)

---

#### `rm` — Remove Files and Directories

```bash
# Remove a file
rm file1.txt

# Remove a directory and all its contents
rm -r practice

# Remove without asking for confirmation
rm -rf practice
```

> **Warning:** `rm` permanently deletes files. There is no recycle bin on Linux servers. Be very careful, especially with `rm -rf`. Always double-check the path before pressing Enter.

---

### Viewing File Contents

#### `cat` — Display Entire File

```bash
cat file1.txt
```

---

#### `less` — View File Page by Page

```bash
less /var/log/syslog
```

- Press **Space** to go to the next page
- Press **b** to go back
- Press **q** to quit

---

#### `head` and `tail` — View Beginning or End of File

```bash
# Show first 10 lines
head file1.txt

# Show last 10 lines
tail file1.txt

# Show last 20 lines
tail -n 20 file1.txt
```

---

### Other Useful Commands

#### `clear` — Clear the Terminal Screen

```bash
clear
```

Or press **Ctrl + L** for the same effect.

---

#### `whoami` — Show Current Username

```bash
whoami
```

**Output:**
```
ubuntu
```

---

#### `man` — Read the Manual for a Command

```bash
man ls
```

Press **q** to exit the manual page.

---

### Quick Reference Table

| Command | What It Does | Example |
|---------|-------------|---------|
| `pwd` | Show current directory | `pwd` |
| `ls` | List files and folders | `ls -la` |
| `cd` | Change directory | `cd practice` |
| `mkdir` | Create a folder | `mkdir myproject` |
| `touch` | Create an empty file | `touch notes.txt` |
| `cp` | Copy a file/folder | `cp file.txt backup.txt` |
| `mv` | Move or rename | `mv old.txt new.txt` |
| `rm` | Delete a file/folder | `rm file.txt` |
| `cat` | Show file contents | `cat file.txt` |
| `less` | View file page-by-page | `less bigfile.log` |
| `head` | Show first N lines | `head -n 5 file.txt` |
| `tail` | Show last N lines | `tail -n 5 file.txt` |
| `clear` | Clear terminal | `clear` |
| `whoami` | Show current user | `whoami` |
| `man` | Read command manual | `man ls` |

---

## 3.4 Step-by-Step Practice

Open the terminal on your Ubuntu VM (`Ctrl + Alt + T`) and execute these commands one by one. Type them manually — do not copy-paste.

### Exercise 1: Navigate and Explore

```bash
# 1. Check where you are
pwd

# 2. List what is in your home directory
ls -la

# 3. Go to the root directory and explore
cd /
ls

# 4. Come back to your home directory
cd ~
pwd
```

### Exercise 2: Create Files and Folders

```bash
# 1. Create a directory called "practice"
mkdir practice

# 2. Move into it
cd practice

# 3. Confirm you are inside the practice directory
pwd

# 4. Create two files
touch file1.txt
touch file2.txt

# 5. List the files to confirm they were created
ls -l
```

### Exercise 3: Copy, Move, and Delete

```bash
# 1. Copy file1.txt to a backup
cp file1.txt file1_backup.txt

# 2. Rename file2.txt to notes.txt
mv file2.txt notes.txt

# 3. List to see the changes
ls -l

# 4. Delete the backup file
rm file1_backup.txt

# 5. Confirm it is gone
ls -l
```

### Exercise 4: View the System

```bash
# 1. See who you are
whoami

# 2. See system information
uname -a

# 3. See disk usage
df -h

# 4. See memory usage
free -h
```

---

## 3.5 Self-Study Prompts

### Prompt 1: Explain the Linux directory structure.

<details>
<summary>Click to see a sample answer</summary>

Linux uses a tree-like directory structure starting from the root `/`. Unlike Windows which uses drive letters (C:\, D:\), everything in Linux lives under a single root.

Key directories:
- `/home` — contains personal folders for each user (like "Users" in Windows)
- `/etc` — stores system-wide configuration files
- `/var` — holds data that changes during system operation (logs, web files)
- `/usr` — contains user programs, libraries, and documentation
- `/tmp` — temporary files that are deleted on reboot
- `/root` — the home directory for the root (administrator) user
- `/bin` and `/sbin` — essential system commands and administration tools

Every user gets a folder under `/home`. For example, a user named "ubuntu" would have `/home/ubuntu` as their home directory, also referred to as `~`.

</details>

### Prompt 2: What is the difference between the `cp` and `mv` commands?

<details>
<summary>Click to see a sample answer</summary>

**`cp` (copy):**
- Creates a duplicate of the file or directory
- The original remains in its location
- After the operation, both the original and the copy exist
- Example: `cp file.txt backup.txt` creates `backup.txt` while keeping `file.txt`

**`mv` (move):**
- Moves a file or directory from one location to another
- The original is removed from its old location
- Also used for renaming files (moving a file to the same directory with a different name)
- Example: `mv file.txt notes.txt` renames `file.txt` to `notes.txt`

**Analogy:** `cp` is like photocopying a document — you get a copy and keep the original. `mv` is like picking up a document and placing it somewhere else — the original location is now empty.

</details>

---

## 3.6 Activity

### Instructions

1. Open the terminal on your Ubuntu VM (`Ctrl + Alt + T`)
2. Create a folder called `practice`
3. Navigate into the `practice` folder
4. Create two files: `file1.txt` and `file2.txt`
5. Run the following command to verify:
   ```bash
   ls -l
   ```
6. Take a screenshot showing the output

### Submission

- **Upload your screenshot**
- **Caption:** `Basic Linux commands executed`

---

## 3.7 Troubleshooting

| Problem | Solution |
|---------|----------|
| "Permission denied" when creating a file | You might be in a system directory. Navigate back to your home directory with `cd ~` and try again |
| "No such file or directory" | Check your spelling. Use `pwd` to confirm your current location and `ls` to see what is available |
| Command not found | Check for typos. Linux commands are case-sensitive: `LS` is not the same as `ls` |
| Accidentally deleted important files | In a VM, you can create snapshots before experimenting. If things go badly wrong, restore the snapshot or reinstall |

---

## 3.8 Summary

In this chapter you learned:

- The Linux directory structure and how it differs from Windows
- How to navigate directories with `pwd`, `ls`, and `cd`
- How to create files (`touch`) and directories (`mkdir`)
- How to copy (`cp`), move/rename (`mv`), and delete (`rm`) files
- How to view file contents with `cat`, `less`, `head`, and `tail`
- Important system commands: `whoami`, `uname -a`, `df -h`, `free -h`

These commands form the foundation for everything else in this module.

**Next:** [Chapter 4 — IP Address Fundamentals](chapter-04-ip-address-fundamentals.md)
