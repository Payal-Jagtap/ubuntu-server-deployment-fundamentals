# Chapter 10 — File Permissions

| | |
|---|---|
| **Objective** | Understand and manage file ownership and permissions using `chmod` and `chown` |
| **Duration** | 25–35 minutes |
| **Difficulty** | Intermediate |
| **Prerequisites** | Chapter 9 completed (understanding of Linux users) |

---

## 10.1 Overview

In the previous chapter, you learned how to create and manage users. Now the question is: **what can each user do?** Linux controls this through a **permission system** that determines who can read, write, and execute every file and directory on the system.

In this chapter, you will learn how to:
- Read and interpret file permission strings
- Use `chmod` to change what actions are allowed on a file
- Use `chown` to change who owns a file
- Apply permissions to secure your Flask application

Understanding permissions is essential for server security. Misconfigured permissions are one of the most common causes of deployment issues ("permission denied" errors) and security vulnerabilities.

---

## 10.2 Understanding the Concepts

### How File Permissions Work

Every file and directory in Linux has three sets of permissions for three categories of users:

```
-rwxr-xr--  1  ubuntu  developers  4096  Jan 15 10:30  app.py
│├─┤├─┤├─┤     │        │
││  │  │       │        └── Group owner
││  │  │       └── User owner
││  │  └── Others (everyone else)
││  └── Group (users in the group)
│└── Owner (the user who owns the file)
└── File type (- = file, d = directory)
```

### The Three Permissions

| Symbol | Permission | For Files | For Directories |
|--------|-----------|-----------|-----------------|
| `r` | **Read** | Can view the file contents | Can list the directory contents (`ls`) |
| `w` | **Write** | Can modify the file | Can create/delete files inside the directory |
| `x` | **Execute** | Can run the file as a program | Can enter the directory (`cd`) |
| `-` | **No permission** | Cannot perform that action | Cannot perform that action |

### The Three User Categories

| Category | Who | Applies To |
|----------|-----|------------|
| **Owner (u)** | The user who owns the file | First `rwx` triplet |
| **Group (g)** | Users in the file's group | Second `rwx` triplet |
| **Others (o)** | Everyone else on the system | Third `rwx` triplet |

### Reading Permission Strings

Let's decode some examples:

**Example 1:**
```
-rwxr-xr--
```

| Part | Meaning |
|------|---------|
| `-` | This is a regular file (not a directory) |
| `rwx` | Owner can read, write, and execute |
| `r-x` | Group can read and execute, but not write |
| `r--` | Others can only read |

**Example 2:**
```
drwxr-x---
```

| Part | Meaning |
|------|---------|
| `d` | This is a directory |
| `rwx` | Owner can read, write, and enter |
| `r-x` | Group can read and enter, but not create files |
| `---` | Others have no access at all |

### Numeric (Octal) Permission System

Each permission has a numeric value:

| Permission | Value |
|-----------|-------|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |
| None (-) | 0 |

You add the values together for each category:

| Combination | Calculation | Value |
|-------------|-------------|-------|
| `rwx` | 4 + 2 + 1 | **7** |
| `rw-` | 4 + 2 + 0 | **6** |
| `r-x` | 4 + 0 + 1 | **5** |
| `r--` | 4 + 0 + 0 | **4** |
| `---` | 0 + 0 + 0 | **0** |

So the permission string `rwxr-xr--` translates to the number **754**:
- Owner: `rwx` = 7
- Group: `r-x` = 5
- Others: `r--` = 4

### Common Permission Numbers

| Number | Permission | Common Use |
|--------|-----------|------------|
| **755** | `rwxr-xr-x` | Directories and executable files. Owner has full control, everyone else can read and execute. |
| **644** | `rw-r--r--` | Regular files. Owner can read and write, everyone else can only read. |
| **700** | `rwx------` | Private directories. Only the owner has access. |
| **600** | `rw-------` | Private files. Only the owner can read and write. |
| **777** | `rwxrwxrwx` | Full access for everyone. **Avoid this** — it is a security risk. |

---

## 10.3 Step-by-Step Instructions

Connect to your Ubuntu VM via SSH from your Windows terminal:

```cmd
ssh ubuntu@192.168.1.50
```

### Part 1: Viewing and Changing Permissions (`chmod`)

#### Step 1: View Current Permissions

```bash
cd ~
touch testfile.txt
mkdir testdir
ls -l
```

**Output:**
```
drwxrwxr-x 2 ubuntu ubuntu 4096 Jan 15 10:30 testdir
-rw-rw-r-- 1 ubuntu ubuntu    0 Jan 15 10:31 testfile.txt
```

Notice:
- `testdir` has permissions `rwxrwxr-x` (775) — owner and group have full access, others can read and enter
- `testfile.txt` has permissions `rw-rw-r--` (664) — owner and group can read/write, others can only read
- Both are owned by `ubuntu` (user) and `ubuntu` (group)

#### Step 2: Change Permissions Using Numeric Notation

```bash
# Give owner full access, group read/execute, others read only
chmod 754 testfile.txt
ls -l testfile.txt
```

**Output:**
```
-rwxr-xr-- 1 ubuntu ubuntu 0 Jan 15 10:31 testfile.txt
```

Let's break down what `754` means:
- `7` (owner) = `rwx` = read + write + execute
- `5` (group) = `r-x` = read + execute
- `4` (others) = `r--` = read only

More examples:

```bash
# Private file — only owner can read and write
chmod 600 testfile.txt
ls -l testfile.txt
# Output: -rw------- 1 ubuntu ubuntu 0 Jan 15 10:31 testfile.txt

# Standard file — owner read/write, everyone else read
chmod 644 testfile.txt
ls -l testfile.txt
# Output: -rw-r--r-- 1 ubuntu ubuntu 0 Jan 15 10:31 testfile.txt

# Executable — owner full, group and others read/execute
chmod 755 testfile.txt
ls -l testfile.txt
# Output: -rwxr-xr-x 1 ubuntu ubuntu 0 Jan 15 10:31 testfile.txt
```

#### Step 3: Change Permissions Using Symbolic Notation

Symbolic notation uses letters instead of numbers — useful for adding or removing specific permissions:

```bash
# Add execute permission for the owner
chmod u+x testfile.txt

# Remove write permission from the group
chmod g-w testfile.txt

# Add read permission for others
chmod o+r testfile.txt

# Set exact permissions for everyone at once
chmod u=rwx,g=rx,o=r testfile.txt
```

**Symbolic notation reference:**

| Symbol | Meaning |
|--------|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (user + group + others) |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

**When to use which:**
- **Numeric (`chmod 755`)** — when you want to set all permissions at once. Fast and precise.
- **Symbolic (`chmod u+x`)** — when you want to change one specific permission without affecting the others.

---

### Part 2: Changing Ownership (`chown`)

#### Step 4: Change File Owner

Make sure you created the `devuser` account from Chapter 9. If not:
```bash
sudo adduser devuser
```

Now change the owner of a file:

```bash
# Change owner to devuser
sudo chown devuser testfile.txt
ls -l testfile.txt
```

**Output:**
```
-rwxr-xr-x 1 devuser ubuntu 0 Jan 15 10:31 testfile.txt
```

The owner changed from `ubuntu` to `devuser`, but the group is still `ubuntu`.

#### Step 5: Change Both Owner and Group

```bash
# Change both owner and group
sudo chown devuser:devuser testfile.txt
ls -l testfile.txt
```

**Output:**
```
-rwxr-xr-x 1 devuser devuser 0 Jan 15 10:31 testfile.txt
```

Now both owner and group are `devuser`.

#### Step 6: Change Ownership Recursively

For directories with files inside, use `-R` to apply changes to everything:

```bash
sudo chown -R devuser:devuser testdir
ls -l
```

**Common `chown` patterns:**

| Command | What It Does |
|---------|-------------|
| `sudo chown devuser file.txt` | Change owner only |
| `sudo chown devuser:devuser file.txt` | Change owner and group |
| `sudo chown :devuser file.txt` | Change group only |
| `sudo chown -R devuser:devuser mydir/` | Change owner/group recursively for a directory |

> **Note:** `chown` always requires `sudo` — regular users cannot change file ownership. This is a security feature. If anyone could change ownership, a malicious user could claim ownership of files they should not have access to.

---

### Part 3: Understanding Permission Errors

#### Step 7: See Permissions in Action

```bash
# Create a file as ubuntu and make it private
touch /tmp/ubuntufile.txt
chmod 600 /tmp/ubuntufile.txt

# Switch to devuser
sudo su - devuser

# Try to read the file
cat /tmp/ubuntufile.txt
```

**Output:**
```
cat: /tmp/ubuntufile.txt: Permission denied
```

`devuser` cannot read the file because:
- The file has permissions `600` (`rw-------`)
- Only the owner (`ubuntu`) can read and write
- `devuser` is "others" — and others have no permissions (`---`)

```bash
# Return to ubuntu
exit
```

This demonstrates exactly why permissions matter — they prevent unauthorized access even between users on the same system.

---

### Part 4: Practical Example — Securing the Flask App

In a production environment, you would set permissions on your deployed application like this:

```bash
# Change ownership of the project to devuser
sudo chown -R devuser:devuser ~/your-flask-app

# Set directory permissions (owner can do everything, others can read/enter)
sudo chmod 755 ~/your-flask-app

# Set file permissions (owner can read/write, others can only read)
sudo find ~/your-flask-app -type f -exec chmod 644 {} \;

# Make the virtual environment's scripts executable
sudo chmod -R 755 ~/your-flask-app/venv/bin/
```

**Why these specific permissions?**

| What | Permission | Why |
|------|-----------|-----|
| Directories | 755 | Owner (devuser) can modify. Nginx (www-data, an "other") needs to read and enter directories to serve files. |
| Regular files | 644 | Owner can edit. Nginx needs to read files to serve them. |
| Executable scripts | 755 | Gunicorn and Python scripts in `venv/bin/` need execute permission to run. |

---

## 10.4 Self-Study Prompts

### Prompt 1: Explain the rwx permission system in Linux.

<details>
<summary>Click to see a sample answer</summary>

Linux uses a three-tiered permission system with three types of permissions applied to three categories of users.

**The three permissions:**
- **r (read):** For files, allows viewing contents. For directories, allows listing contents.
- **w (write):** For files, allows modifying contents. For directories, allows creating or deleting files inside.
- **x (execute):** For files, allows running the file as a program. For directories, allows entering (using `cd`).

**The three categories:**
- **User (u):** The owner of the file — the user who created it.
- **Group (g):** Members of the file's group. Linux groups allow multiple users to share permissions.
- **Others (o):** Everyone else on the system.

**How to read a permission string:**
```
-rwxr-xr-- means:
  - Regular file
  - Owner: read + write + execute (rwx = 7)
  - Group: read + execute (r-x = 5)
  - Others: read only (r-- = 4)
  Numeric: 754
```

**Key principle:** Permissions are checked in order: owner first, then group, then others. The first match applies. If you are the owner, the owner permissions are used, even if the group or others permissions are more permissive.

</details>

### Prompt 2: What is the difference between `chmod` and `chown`?

<details>
<summary>Click to see a sample answer</summary>

**`chmod` (change mode)** — Changes **what actions** can be performed on a file.

It modifies the read, write, and execute permission bits. It answers the question: "What can the owner/group/others **do** with this file?"

```bash
chmod 755 file.txt    # Owner: rwx, Group: r-x, Others: r-x
chmod u+x script.sh   # Add execute permission for the owner
```

**`chown` (change owner)** — Changes **who owns** a file.

It modifies the user and/or group ownership. It answers the question: "**Who** is the owner and group of this file?"

```bash
chown devuser file.txt            # Change owner to devuser
chown devuser:devuser file.txt    # Change owner AND group to devuser
```

**Analogy:** Think of a room in a building.
- `chown` is like transferring the room's lease to a different tenant — changing **who** the room belongs to.
- `chmod` is like changing the room's lock — deciding **who can enter, who can write on the whiteboard, and who can move furniture**.

**Key difference:** `chown` requires `sudo` (admin privileges) because letting users change file ownership would be a security risk. `chmod` can be done by the file's owner without `sudo`.

</details>

---

## 10.5 Activity

### Instructions

1. SSH into your Ubuntu VM
2. Create a test file:
   ```bash
   touch ~/permtest.txt
   ```
3. Change the file's permissions to `755`:
   ```bash
   chmod 755 ~/permtest.txt
   ```
4. Change the file's owner to `devuser`:
   ```bash
   sudo chown devuser:devuser ~/permtest.txt
   ```
5. Verify the changes:
   ```bash
   ls -l ~/permtest.txt
   ```
6. Take a screenshot showing the `ls -l` output with the new owner and permissions

### Submission

- **Upload your screenshot**
- **Caption:** `File permissions configured`

---

## 10.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| `chown: Operation not permitted` | You need `sudo`. Run: `sudo chown devuser:devuser filename` |
| `chmod: changing permissions of 'file': Operation not permitted` | You are not the file's owner and are not using `sudo`. Either use `sudo chmod` or switch to the file's owner. |
| Permission denied when accessing a file you should be able to read | Check the permissions of all **parent directories**. You need at least `x` (execute) permission on every directory in the path to reach the file. |
| Changed permissions but the application still cannot access files | Make sure you changed permissions on the correct files **and** directories. Also check that the user running the application is the owner or in the correct group. |

---

## 10.7 Summary

In this chapter you learned:

- The three types of file permissions: read (`r`), write (`w`), execute (`x`)
- The three user categories: owner, group, others
- How to read permission strings (like `rwxr-xr--` or `754`)
- How to change permissions with `chmod` (both numeric and symbolic notation)
- How to change file ownership with `chown`
- How to secure a deployed application with proper ownership and permissions

These skills are important for:
- Securing your Flask application files
- Ensuring Nginx and Gunicorn can access the files they need
- Following the principle of least privilege
- Debugging "permission denied" errors during deployment

---

## 10.8 Module Completion

Congratulations! You have completed all 10 chapters of the **Ubuntu Server Deployment Fundamentals** module.

### What You Have Accomplished

| Chapter | What You Built/Learned |
|---------|----------------------|
| 1 | Installed VirtualBox — your virtualization platform |
| 2 | Installed Ubuntu — your Linux environment with desktop and terminal |
| 3 | Mastered basic Linux commands — your foundation for server management |
| 4 | Understood IP addresses — how devices find each other on networks |
| 5 | Learned about ports — how services are identified on a single machine |
| 6 | Configured SSH and firewall — secure remote access and network security |
| 7 | Deployed Flask with Gunicorn — your application running in production mode |
| 8 | Configured Nginx — professional reverse proxy serving your application |
| 9 | Managed users — creating accounts and controlling access |
| 10 | Configured file permissions — securing files with ownership and rwx |

### Your Final Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Windows Host Machine                                           │
│                                                                 │
│   Browser ─────► http://192.168.1.50                            │
│   SSH Client ──► ssh ubuntu@192.168.1.50                        │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  VirtualBox VM — Ubuntu                                  │   │
│   │                                                         │   │
│   │   UFW Firewall: Ports 22 (SSH), 80 (HTTP) open          │   │
│   │                                                         │   │
│   │   Nginx (Port 80) ──► Gunicorn (Port 8000) ──► Flask    │   │
│   │                                                         │   │
│   │   Users: ubuntu (admin), devuser (app owner)            │   │
│   │   File permissions properly configured                  │   │
│   │                                                         │   │
│   │   Application cloned from Git repository                │   │
│   │   Python virtual environment isolating dependencies     │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Recommended Next Steps

Now that you have completed this module, consider exploring:

1. **HTTPS with SSL/TLS** — Secure your site with Let's Encrypt certificates
2. **Systemd services** — Create a service file so Gunicorn starts automatically on boot
3. **Domain names** — Point a real domain to your server
4. **Database setup** — Add PostgreSQL or MySQL to your deployment
5. **CI/CD pipelines** — Automate deployments using GitHub Actions
6. **Docker** — Containerize your application for easier deployment
7. **Cloud deployment** — Deploy on AWS EC2, DigitalOcean, or Azure

You now have the foundational knowledge to understand and work with any of these technologies. Well done!
