# Chapter 9 — User Management

| | |
|---|---|
| **Objective** | Understand Linux user types and create and manage user accounts |
| **Duration** | 20–25 minutes |
| **Difficulty** | Intermediate |
| **Prerequisites** | Chapter 3 completed (basic Linux commands), Chapter 6 completed (SSH access) |

---

## 9.1 Overview

Linux is a **multi-user operating system** — multiple users can use the same server, each with their own files, settings, and access levels. In a production environment, you would not run your web application as the root (admin) user. Instead, you create dedicated users with limited permissions to reduce security risk.

In this chapter, you will learn how to:
- Understand the different types of Linux users
- Use `sudo` for administrative tasks
- Create new user accounts
- Switch between users
- Manage group memberships

---

## 9.2 Understanding the Concepts

### Linux User Types

| User Type | Description | Example |
|-----------|-------------|---------|
| **Root** | The superuser/administrator. Has unlimited access to everything. | `root` |
| **Regular User** | A normal user account with limited access. Can only modify their own files. | `ubuntu`, `devuser` |
| **System User** | Created automatically for running services. Cannot log in interactively. | `www-data` (Nginx), `nobody` |

### Why Not Run Everything as Root?

Running applications as root is dangerous because:

1. **If the application is compromised**, the attacker gets root access — they can do anything to the server
2. **Mistakes are amplified** — a typo like `rm -rf /` as root deletes everything; as a regular user, it only deletes your files
3. **Principle of least privilege** — each user/process should have only the minimum permissions needed to do its job

In production:
- Nginx runs as the `www-data` user
- Your Flask app should run as a dedicated user (e.g., `devuser`)
- You use `sudo` only when you specifically need admin privileges

### The `sudo` Command

Regular users cannot perform administrative tasks. The `sudo` command (Super User DO) temporarily elevates your privileges:

```bash
# This fails — regular users cannot install software
apt install nginx

# This works — sudo runs the command with root privileges
sudo apt install nginx
```

When you run `sudo`, you are asked for **your own password** (not the root password). Not all users have sudo access — only those added to the `sudo` group.

### Users and Groups

Every user in Linux belongs to at least one **group**. Groups are a way to manage permissions for multiple users at once.

```
Groups on a typical server:
┌──────────────────────────────────────────┐
│  sudo group                              │
│  Members: ubuntu                         │
│  (Can run administrative commands)       │
├──────────────────────────────────────────┤
│  www-data group                          │
│  Members: www-data, devuser              │
│  (Can access web server files)           │
├──────────────────────────────────────────┤
│  devuser group                           │
│  Members: devuser                        │
│  (Default group for devuser)             │
└──────────────────────────────────────────┘
```

When you create a user with `adduser`, Linux automatically creates a group with the same name and assigns the user to it. A user can belong to multiple groups.

### Where User Information Is Stored

| File | What It Contains |
|------|------------------|
| `/etc/passwd` | User account information (username, user ID, home directory, shell) |
| `/etc/shadow` | Encrypted passwords (only readable by root) |
| `/etc/group` | Group definitions and memberships |

---

## 9.3 Step-by-Step Instructions

Connect to your Ubuntu VM via SSH from your Windows terminal:

```cmd
ssh ubuntu@192.168.1.50
```

### Step 1: See Your Current User

```bash
whoami
```

**Output:**
```
ubuntu
```

Check which groups you belong to:

```bash
groups
```

**Output:**
```
ubuntu adm sudo ...
```

This shows you are in the `sudo` group, which means you can run administrative commands.

Get detailed information:

```bash
id
```

**Output:**
```
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),27(sudo)...
```

- `uid=1000` — your user ID
- `gid=1000` — your primary group ID
- The rest are additional groups you belong to

### Step 2: Create a New User

```bash
sudo adduser devuser
```

You will be prompted to:
1. **Enter new password:** Choose a password for the new user
2. **Retype new password:** Confirm the password
3. **Full Name:** Type a name or press Enter to skip
4. **Room Number, Work Phone, Home Phone, Other:** Press Enter to skip each one
5. **Is the information correct?** Type `Y` and press Enter

**Output:**
```
Adding user `devuser' ...
Adding new group `devuser' (1001) ...
Adding new user `devuser' (1001) with group `devuser' ...
Creating home directory `/home/devuser' ...
Copying files from `/etc/skel' ...
```

The `adduser` command:
- Creates the user account
- Creates a home directory at `/home/devuser`
- Creates a group with the same name (`devuser`)
- Copies default configuration files to the home directory

### Step 3: Verify the User Was Created

```bash
# Look up the new user's entry
cat /etc/passwd | grep devuser

# Check the home directory was created
ls /home/
```

**Expected output:**
```
devuser:x:1001:1001::/home/devuser:/bin/bash
```

This shows:
- Username: `devuser`
- User ID: `1001`
- Home directory: `/home/devuser`
- Shell: `/bin/bash`

### Step 4: Switch to the New User

```bash
sudo su - devuser
```

- `su` — switch user
- `-` — also load the user's environment (home directory, shell settings)
- `devuser` — the user to switch to

Your prompt changes to:
```
devuser@ubuntu-vm:~$
```

You are now acting as `devuser`. Verify:

```bash
whoami
pwd
```

**Output:**
```
devuser
/home/devuser
```

### Step 5: Return to Your Original User

```bash
exit
```

You are back as `ubuntu`. Your prompt returns to:
```
ubuntu@ubuntu-vm:~$
```

### Step 6: Give sudo Access (Optional)

By default, a new user cannot use `sudo`. To grant sudo access:

```bash
sudo usermod -aG sudo devuser
```

- `usermod` — modify a user account
- `-aG sudo` — **a**ppend to the **G**roup called `sudo`

> **Note:** The user must log out and back in for group changes to take effect.

Verify the group was added:

```bash
groups devuser
```

**Output:**
```
devuser : devuser sudo
```

### Step 7: Change a User's Password

```bash
sudo passwd devuser
```

You will be prompted to enter and confirm a new password.

### User Management Commands Reference

| Command | What It Does |
|---------|-------------|
| `whoami` | Show current username |
| `id` | Show user ID, group ID, and all groups |
| `groups username` | Show which groups a user belongs to |
| `sudo adduser username` | Create a new user |
| `sudo deluser username` | Delete a user (keeps home directory) |
| `sudo deluser --remove-home username` | Delete a user and their home directory |
| `sudo passwd username` | Change a user's password |
| `sudo usermod -aG groupname username` | Add a user to a group |
| `sudo su - username` | Switch to another user |
| `exit` | Return to the previous user |

---

## 9.4 Self-Study Prompts

### Prompt 1: What is the difference between the root user and a regular user?

<details>
<summary>Click to see a sample answer</summary>

The **root user** is the superuser or administrator of a Linux system. Root has unrestricted access to every file, command, and service on the system. The root user can:
- Install and remove any software
- Create, modify, or delete any file — including system files
- Start and stop any service
- Create and delete user accounts
- Change any system configuration

A **regular user** has limited access. They can only:
- Read, write, and execute files they own or have permission to access
- Run programs available to normal users
- Modify files in their own home directory

Regular users can temporarily gain root privileges using `sudo`, but only if they have been added to the `sudo` group. This is a security measure — instead of always being root (which is risky), you operate as a normal user and only escalate when needed.

**Best practice:** Never log in as root directly. Use a regular user account and `sudo` when necessary. This way, dangerous commands require an extra deliberate step (typing `sudo`), which helps prevent accidental damage.

</details>

### Prompt 2: Why do we create separate users for different applications?

<details>
<summary>Click to see a sample answer</summary>

Creating separate users for different applications follows the **principle of least privilege** — each process should only have access to the resources it actually needs. Here is why this matters:

1. **Security isolation:** If your Flask app runs as `devuser` and gets hacked, the attacker only has access to `devuser`'s files. They cannot touch system files, other users' data, or other applications. If the app ran as root, a compromise would give the attacker full control of the entire server.

2. **Damage limitation:** If an application has a bug that deletes files, the damage is limited to that user's files. Other users and system files remain safe.

3. **Accountability:** You can track which user (and therefore which application or person) made changes to files, started processes, or generated log entries.

4. **Resource management:** You can set resource limits (CPU, memory, disk) per user, preventing one application from consuming everything.

Real-world example:
- Nginx runs as `www-data` — it only needs to read web files and listen on ports 80/443
- Your Flask app runs as `devuser` — it only needs access to the project directory
- The database runs as `postgres` — it only needs access to its data directory

If any one of these gets compromised, the others remain unaffected.

</details>

---

## 9.5 Activity

### Instructions

1. SSH into your Ubuntu VM
2. Create a new user called `devuser`:
   ```bash
   sudo adduser devuser
   ```
3. Switch to the new user:
   ```bash
   sudo su - devuser
   ```
4. Run:
   ```bash
   whoami
   ```
5. Return to your original user:
   ```bash
   exit
   ```
6. Take a screenshot showing the user creation and the `whoami` output

### Submission

- **Upload your screenshot**
- **Caption:** `User created and verified`

---

## 9.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| `adduser: command not found` | Use `sudo useradd -m username` instead (different command, fewer prompts). Or install: `sudo apt install adduser`. |
| `adduser: Only root may add a user` | You forgot `sudo`. Run: `sudo adduser devuser` |
| Created user cannot run `sudo` | Add them to the sudo group: `sudo usermod -aG sudo devuser`. The user must log out and back in. |
| `su: Authentication failure` | You entered the wrong password for the target user. Use `sudo su - devuser` instead (uses your password, not devuser's). |
| Forgot a user's password | Reset it: `sudo passwd username` |

---

## 9.7 Summary

In this chapter you learned:

- The three types of Linux users: root, regular, and system
- Why running applications as root is a security risk
- How `sudo` temporarily elevates privileges
- How to create users with `adduser`
- How to switch between users with `su`
- How to manage group memberships with `usermod`

In the next chapter, you will learn how to control **what each user can do** with files through the Linux permission system.

**Next:** [Chapter 10 — File Permissions](chapter-10-file-permissions.md)
