# Chapter 6 — Firewall, SSH Setup and Remote Login

| | |
|---|---|
| **Objective** | Secure the server with firewall rules and connect remotely using SSH |
| **Duration** | 30–40 minutes |
| **Difficulty** | Intermediate |
| **Prerequisites** | Chapters 4 and 5 completed (understanding of IP addresses and ports) |

---

## 6.1 Overview

Up to this point, you have been working directly inside the VirtualBox window to interact with your Ubuntu VM. In a real production environment, servers are located in data centers — you cannot walk up and type on their keyboards. Instead, you connect **remotely** using a protocol called **SSH** (Secure Shell).

In this chapter you will:

1. Install the SSH server on Ubuntu
2. Set up a firewall to control which network traffic is allowed
3. Connect to your Ubuntu VM from your Windows machine using SSH

After this chapter, you will be able to manage your server from the comfort of your Windows terminal — exactly how professionals manage servers in the real world.

---

## 6.2 Understanding the Concepts

### What Is SSH?

**SSH (Secure Shell)** is a network protocol that allows you to securely log in to a remote computer and execute commands on it. Everything you type is **encrypted** — even if someone intercepts the data on the network, they cannot read it.

```
┌───────────────────┐        Encrypted Connection        ┌───────────────────┐
│                   │  ◄──────────────────────────────►  │                   │
│  Your Windows PC  │         SSH (Port 22)              │  Ubuntu VM        │
│  (SSH Client)     │                                    │  (SSH Server)     │
│                   │                                    │                   │
└───────────────────┘                                    └───────────────────┘
   You type here                                            Commands run here
```

**Before SSH existed**, people used **Telnet** to connect to remote computers. Telnet sends everything (including passwords) as plain text — anyone on the same network could see your password. SSH replaced Telnet by encrypting all communication.

### How SSH Authentication Works

When you run `ssh ubuntu@192.168.1.50`, here is what happens:

```
Step 1: Your computer (client) connects to port 22 on the server
Step 2: Server and client negotiate encryption
Step 3: An encrypted tunnel is established
Step 4: Server asks for your username and password
Step 5: You type your password (sent through the encrypted tunnel)
Step 6: Server verifies your credentials
Step 7: You are logged in — your terminal is now connected to the server
```

### What Is a Firewall?

A **firewall** is a security system that controls what network traffic is allowed in and out of your server. Think of it as a security guard at the entrance of a building.

```
         Incoming Traffic
              │
              ▼
     ┌──────────────────┐
     │     FIREWALL      │
     │                    │
     │  Port 22 ──► ALLOW │   (SSH connections permitted)
     │  Port 80 ──► ALLOW │   (Web traffic permitted)
     │  Port 443 ─► ALLOW │   (HTTPS traffic permitted)
     │  Port 3306 ► DENY  │   (Database not exposed)
     │  All else ─► DENY  │   (Everything else blocked)
     │                    │
     └──────────────────┘
              │
              ▼
     ┌──────────────────┐
     │     SERVER        │
     │  (Ubuntu)         │
     └──────────────────┘
```

**Without a firewall:**
- Every service running on the server is accessible from the network
- Attackers can probe all ports looking for vulnerabilities
- Even internal services (like databases) are exposed

**With a firewall:**
- Only explicitly allowed ports are accessible
- Unwanted traffic is dropped before reaching any service
- The server is protected from unauthorized access

### UFW — Uncomplicated Firewall

Ubuntu includes a firewall tool called **UFW** (Uncomplicated Firewall). It is a simplified interface for managing firewall rules. Instead of writing complex iptables rules, you can use simple commands like:

```bash
sudo ufw allow 22     # Allow SSH
sudo ufw allow 80     # Allow web traffic
sudo ufw deny 3306    # Block MySQL from external access
```

---

## 6.3 Step-by-Step Instructions

### Part 1: Install and Configure SSH Server

Open the terminal on your Ubuntu VM (`Ctrl + Alt + T`).

#### Step 1: Update Package Lists

Always update before installing new software:

```bash
sudo apt update
```

This downloads the latest package information so `apt` knows what versions are available.

#### Step 2: Install OpenSSH Server

```bash
sudo apt install openssh-server -y
```

- `openssh-server` — the SSH server package
- `-y` — automatically answer "yes" to installation prompts

#### Step 3: Verify SSH Is Running

```bash
sudo systemctl status ssh
```

You should see output containing:
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded
     Active: active (running)
```

If it says `active (running)`, SSH is ready. Press `q` to exit the status view.

#### Step 4: Check SSH Is Listening on Port 22

```bash
sudo ss -tuln | grep 22
```

You should see:
```
tcp   LISTEN   0   128   0.0.0.0:22   0.0.0.0:*
```

This confirms SSH is listening on port 22 on all network interfaces.

---

### Part 2: Configure the Firewall (UFW)

#### Step 5: Allow SSH Through the Firewall

Before enabling the firewall, you **must** allow SSH first. If you enable the firewall without allowing SSH, you will lock yourself out of the server.

```bash
sudo ufw allow OpenSSH
```

**Output:**
```
Rules updated
Rules updated (v6)
```

This creates a rule allowing incoming connections on port 22 (the default SSH port).

#### Step 6: Enable the Firewall

```bash
sudo ufw enable
```

**You will see a warning:**
```
Command may disrupt existing SSH connections. Proceed with operation (y|n)?
```

Type `y` and press Enter.

**Output:**
```
Firewall is active and enabled on system startup
```

#### Step 7: Verify Firewall Status

```bash
sudo ufw status verbose
```

**Expected output:**
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp (OpenSSH)           ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)
```

**What this means:**

| Setting | Meaning |
|---------|---------|
| `Default: deny (incoming)` | All incoming traffic is **blocked** unless explicitly allowed |
| `Default: allow (outgoing)` | All outgoing traffic from the server is **allowed** (the server can reach the internet) |
| `22/tcp (OpenSSH) ALLOW IN` | Port 22 is open for incoming SSH connections |

> **Important:** Later, when you set up Nginx (Chapter 8), you will also need to allow port 80:
> ```bash
> sudo ufw allow 'Nginx HTTP'
> ```
> But do not do this now — we will add it in Chapter 8.

#### Useful UFW Commands Reference

| Command | What It Does |
|---------|-------------|
| `sudo ufw status` | Show current firewall rules |
| `sudo ufw allow 80` | Allow incoming traffic on port 80 |
| `sudo ufw deny 3306` | Block incoming traffic on port 3306 |
| `sudo ufw delete allow 80` | Remove the rule that allows port 80 |
| `sudo ufw disable` | Turn off the firewall |
| `sudo ufw reset` | Reset all rules to default |

---

### Part 3: Connect via SSH from Windows

#### Step 8: Find Your VM's IP Address

On your Ubuntu VM, run:

```bash
ip a
```

Look for the `inet` address on your network interface (usually `enp0s3`). For example: `192.168.1.50`.

> **Reminder:** If your VM is using NAT mode and shows `10.0.2.15`, you need to either:
> - Switch to **Bridged Adapter** (recommended — see Chapter 4), or
> - Set up **Port Forwarding** in VirtualBox: VM Settings > Network > Advanced > Port Forwarding > Add Rule:
>   - Host Port: `2222`
>   - Guest Port: `22`
>   - Then connect with: `ssh -p 2222 ubuntu@127.0.0.1`

#### Step 9: Open Windows Terminal or Command Prompt

On your **Windows** machine:
- Press `Windows + R`, type `cmd`, and press Enter
- Or open **Windows Terminal** or **PowerShell**

#### Step 10: Connect via SSH

```cmd
ssh ubuntu@192.168.1.50
```

Replace:
- `ubuntu` with your Ubuntu username
- `192.168.1.50` with your VM's actual IP address

#### Step 11: Accept the Host Key

The first time you connect, you will see:

```
The authenticity of host '192.168.1.50 (192.168.1.50)' can't be established.
ECDSA key fingerprint is SHA256:AbCdEf1234567890...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter. This adds the server to your list of known hosts so you will not be asked again.

#### Step 12: Enter Your Password

```
ubuntu@192.168.1.50's password:
```

Type your Ubuntu password (nothing will appear on screen — this is normal) and press Enter.

#### Step 13: You Are In!

You should see the Ubuntu terminal prompt:

```
ubuntu@ubuntu-vm:~$
```

You are now connected to your Ubuntu VM **from your Windows machine**. Every command you type here executes on the server.

#### Step 14: Verify the Connection

```bash
hostname
```

**Output:**
```
ubuntu-vm
```

This confirms you are on the server, not on your Windows machine.

Try a few more commands to confirm:

```bash
whoami
uname -a
ip a
```

#### Step 15: Disconnect

To exit the SSH session and return to your Windows terminal:

```bash
exit
```

Or press `Ctrl + D`.

---

## 6.4 Self-Study Prompts

### Prompt 1: Explain the SSH login process.

<details>
<summary>Click to see a sample answer</summary>

The SSH login process works as follows:

1. **Client initiates connection:** When you run `ssh ubuntu@192.168.1.50`, your SSH client (on Windows) attempts to connect to port 22 on the server.

2. **Key exchange:** The client and server negotiate encryption algorithms and exchange cryptographic keys. This establishes a secure, encrypted channel — all data from this point forward is encrypted.

3. **Host verification:** The first time you connect, the client does not recognize the server and shows you its "fingerprint" (a unique identifier). You verify it is correct and type "yes". The server's identity is saved so future connections skip this step.

4. **Authentication:** The server asks for your username and password. You type your password, which is sent through the encrypted channel (so no one on the network can see it).

5. **Session established:** If the credentials are correct, the server grants access. Your terminal now acts as if you are sitting at the server's keyboard. Every command you type is sent over the encrypted connection, executed on the server, and the results are sent back.

6. **Session termination:** When you type `exit` or press Ctrl+D, the encrypted connection is closed and you return to your local terminal.

</details>

### Prompt 2: Why are firewall rules necessary?

<details>
<summary>Click to see a sample answer</summary>

Firewall rules are necessary for several reasons:

1. **Limit attack surface:** A server may run many services (SSH, web server, database, etc.). Without a firewall, all of these are accessible from the network. By default-denying all incoming traffic and only allowing specific ports, you ensure that only intended services are reachable.

2. **Protect internal services:** Some services like databases (MySQL on port 3306, PostgreSQL on port 5432) should only be accessed by applications on the same server, not from the internet. A firewall blocks external access to these ports while still allowing local connections.

3. **Prevent unauthorized access:** Automated bots constantly scan the internet looking for open ports to exploit. A firewall reduces the number of ports visible to these scanners, making the server a smaller target.

4. **Compliance and best practices:** Security standards (like PCI-DSS, SOC 2) require firewall configurations as a basic security measure.

5. **Controlled access:** You can create rules that only allow certain IP addresses to connect to certain ports. For example, allowing SSH only from your office IP.

The key principle is **deny by default, allow by exception** — block everything first, then explicitly open only what is needed.

</details>

---

## 6.5 Activity

### Instructions

1. Install OpenSSH server on your Ubuntu VM
2. Configure UFW to allow SSH connections
3. Enable the firewall
4. From your **Windows** terminal, SSH into the Ubuntu VM:
   ```cmd
   ssh ubuntu@<your_vm_ip>
   ```
5. Once connected, run:
   ```bash
   hostname
   ```
6. Take a screenshot showing the SSH session and the hostname output

### Submission

- **Upload your screenshot**
- **Caption:** `SSH connection established`

---

## 6.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| `ssh: connect to host ... port 22: Connection refused` | SSH server is not running. On the VM, run: `sudo systemctl start ssh` |
| `ssh: connect to host ... port 22: Connection timed out` | Network issue. Make sure: (1) VM is using Bridged Adapter, (2) you have the correct IP, (3) firewall allows SSH. |
| `Permission denied (publickey,password)` | Incorrect password. Remember: nothing appears when you type it. Try again carefully. If it keeps failing, check your username. |
| `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` | The server's key changed (perhaps you recreated the VM). Fix it by running on Windows: `ssh-keygen -R 192.168.1.50` (use your VM's IP). |
| `ssh` command not found on Windows | Windows 10/11 includes SSH by default. If missing, install the "OpenSSH Client" feature from Windows Settings > Apps > Optional Features. |
| Locked out after enabling firewall | If you can still access the VM through VirtualBox, log in directly and run: `sudo ufw allow OpenSSH` then `sudo ufw reload`. |
| Everything worked but connection is very slow | This is normal in VirtualBox. You can improve performance by allocating more resources to the VM or using NAT mode with port forwarding instead of Bridged Adapter. |

---

## 6.7 Summary

In this chapter you:

- Installed the OpenSSH server on Ubuntu
- Learned what a firewall is and why it is important
- Configured UFW to allow SSH traffic while blocking everything else by default
- Connected to your Ubuntu VM remotely from Windows using SSH
- Verified the connection by running `hostname`

From this point forward, you can manage your server entirely through SSH. You no longer need to use the VirtualBox window — in fact, using SSH is preferred because:
- You can copy-paste commands more easily
- Your Windows terminal has better text support
- This is how you would manage a real production server

**Next:** [Chapter 7 — Deploying Flask Application from Git](chapter-07-deploying-flask-from-git.md)
