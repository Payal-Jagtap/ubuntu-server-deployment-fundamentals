# Chapter 5 — Ports and Network Communication

| | |
|---|---|
| **Objective** | Understand network ports and learn how to check open ports on Windows and Ubuntu |
| **Duration** | 20–30 minutes |
| **Difficulty** | Beginner |
| **Prerequisites** | Chapter 4 completed (understanding of IP addresses) |

---

## 5.1 Overview

In the previous chapter, you learned that IP addresses identify devices on a network. But a single computer can run **many services at the same time** — a web server, an SSH server, a database, and more. How does the computer know which incoming data should go to which service?

The answer is **ports**. Ports are like numbered doors on a building (the computer). Each service listens on a specific door number, and incoming data is directed to the correct service based on the port number it is addressed to.

Understanding ports is essential before you configure firewalls (Chapter 6), set up SSH (Chapter 6), and deploy web applications (Chapters 7 and 8).

---

## 5.2 Understanding the Concepts

### What Is a Port?

A **port** is a number (from 0 to 65535) that identifies a specific service or application running on a computer. When data travels across a network, it is addressed to both an **IP address** (which computer) and a **port number** (which service on that computer).

**Analogy — The Apartment Building:**

Think of a computer as an apartment building:
- The **IP address** is the street address of the building (e.g., 192.168.1.50)
- The **port** is the apartment number (e.g., Apartment 80)

If someone sends a letter to "192.168.1.50, Port 80", the computer knows to deliver that letter to the **web server** (which lives in Apartment 80).

```
Data Packet: "Go to 192.168.1.50:80"

     ┌──────────────────────────────────────┐
     │   Computer at 192.168.1.50           │
     │                                      │
     │   Port 22  ──► SSH Server            │
     │   Port 80  ──► Nginx Web Server  ◄───┼── This one!
     │   Port 443 ──► HTTPS Server          │
     │   Port 5432──► PostgreSQL Database   │
     │   Port 8000──► Gunicorn (Flask App)  │
     │                                      │
     └──────────────────────────────────────┘
```

### Port Number Ranges

| Range | Name | Description |
|-------|------|-------------|
| 0–1023 | **Well-Known Ports** | Reserved for standard services (HTTP, SSH, etc.). Require root/admin privileges to use. |
| 1024–49151 | **Registered Ports** | Used by software applications. Can be assigned by IANA upon request. |
| 49152–65535 | **Dynamic/Private Ports** | Used temporarily by the operating system for outgoing connections. |

### Common Ports You Need to Know

These are the ports you will encounter in this module:

| Port | Service | Protocol | What It Does |
|------|---------|----------|-------------|
| **22** | SSH | TCP | Secure remote login to servers |
| **80** | HTTP | TCP | Unencrypted web traffic (what Nginx will listen on) |
| **443** | HTTPS | TCP | Encrypted web traffic (secure websites) |
| **8000** | Custom | TCP | Often used by Gunicorn/Flask during development |
| **3306** | MySQL | TCP | MySQL database connections |
| **5432** | PostgreSQL | TCP | PostgreSQL database connections |
| **53** | DNS | TCP/UDP | Domain name resolution (translating google.com to an IP) |
| **25** | SMTP | TCP | Email sending |

### How a Web Request Works (Port in Action)

When you type `http://192.168.1.50` in your browser:

```
Step 1: Browser creates a request
        Destination: 192.168.1.50:80 (port 80 is default for HTTP)

Step 2: Request travels across the network to the server

Step 3: Server receives data on port 80
        Nginx is listening on port 80 → receives the request

Step 4: Nginx forwards to Gunicorn on port 8000

Step 5: Flask application processes the request and sends response back
```

### TCP vs UDP

Ports use two main transport protocols:

| Protocol | Full Name | Characteristics | Used For |
|----------|-----------|-----------------|----------|
| **TCP** | Transmission Control Protocol | Reliable, ordered, error-checked delivery. Connection-based. | Web (HTTP/HTTPS), SSH, email, file transfer |
| **UDP** | User Datagram Protocol | Fast, no guarantee of delivery. Connectionless. | DNS lookups, video streaming, gaming |

For this module, almost everything uses **TCP**.

### The Socket: IP + Port

When an IP address and port are combined, it forms a **socket**:

```
192.168.1.50:80
  IP address  : Port
     │            │
     └── Which    └── Which service
         computer      on that computer
```

A socket uniquely identifies a specific service on a specific machine on the network.

---

## 5.3 Step-by-Step Instructions

### Checking Open Ports on Windows

Open **Command Prompt** (press `Windows + R`, type `cmd`, press Enter) and run:

```cmd
netstat -ano
```

**What the flags mean:**
- `-a` — Show all connections and listening ports
- `-n` — Show addresses and port numbers in numerical form
- `-o` — Show the process ID (PID) associated with each connection

**Sample output:**
```
Proto  Local Address          Foreign Address        State           PID
TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       1044
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
TCP    192.168.1.10:50234     142.250.190.14:443     ESTABLISHED     8764
TCP    192.168.1.10:50235     13.107.42.14:443       ESTABLISHED     12340
```

**Reading the output:**

| Column | Meaning |
|--------|---------|
| **Proto** | Protocol (TCP or UDP) |
| **Local Address** | Your computer's IP and the port being used (e.g., `0.0.0.0:135` means port 135 on all interfaces) |
| **Foreign Address** | The remote computer's IP and port |
| **State** | Connection state (see below) |
| **PID** | Process ID — which program is using this port |

**Connection states:**

| State | Meaning |
|-------|---------|
| `LISTENING` | The port is open and waiting for incoming connections (a service is running) |
| `ESTABLISHED` | An active connection exists between your computer and a remote computer |
| `TIME_WAIT` | The connection was recently closed and is waiting for cleanup |
| `CLOSE_WAIT` | The remote side has closed the connection; your side is cleaning up |

To see only **listening** ports (services waiting for connections):

```cmd
netstat -ano | findstr LISTENING
```

To find out which program is using a specific port (e.g., port 80):

```cmd
netstat -ano | findstr :80
```

---

### Checking Open Ports on Ubuntu

Open the terminal on your Ubuntu VM (`Ctrl + Alt + T`) and run:

```bash
sudo ss -tuln
```

**What the flags mean:**
- `-t` — Show TCP ports
- `-u` — Show UDP ports
- `-l` — Show only listening (open) ports
- `-n` — Show port numbers instead of service names

**Sample output:**
```
Netid  State    Recv-Q  Send-Q   Local Address:Port    Peer Address:Port
tcp    LISTEN   0       128      0.0.0.0:22             0.0.0.0:*
tcp    LISTEN   0       511      0.0.0.0:80             0.0.0.0:*
udp    UNCONN   0       0        127.0.0.53%lo:53       0.0.0.0:*
```

**Reading the output:**

| Column | Meaning |
|--------|---------|
| **Netid** | Protocol (tcp or udp) |
| **State** | `LISTEN` means the port is open and accepting connections |
| **Local Address:Port** | The address and port the service is listening on |
| **Peer Address:Port** | `*` means accepting from any remote address |

From the example:
- Port **22** is open — **SSH server** is running
- Port **80** is open — **Nginx web server** is running
- Port **53** on localhost — **DNS resolver** is running

**Alternative command:** You can also use `netstat` on Ubuntu:

```bash
sudo netstat -tuln
```

The output format is slightly different but provides the same information.

---

### Practical Exercise: See a Port in Action

Let's start a simple listening service and verify the port is open:

```bash
# Start a simple Python HTTP server on port 9999
python3 -m http.server 9999 &

# Check if port 9999 is now listening
sudo ss -tuln | grep 9999

# Stop the server
kill %1
```

You should see port 9999 appear in the listening ports list, then disappear after you stop the server.

---

## 5.4 Self-Study Prompts

### Prompt 1: Explain network ports using a real-world analogy.

<details>
<summary>Click to see a sample answer</summary>

Think of a large office building on a city street.

- The **street address** of the building is like the **IP address** — it tells you which building to go to.
- The **office suite numbers** inside the building are like **ports** — they tell you which specific office (service) to visit.

For example:
- Suite 22 has the "Secure Shell Consulting" office (SSH on port 22)
- Suite 80 has the "Web Services" office (HTTP on port 80)
- Suite 443 has the "Secure Web Services" office (HTTPS on port 443)
- Suite 8000 has the "Python Flask Workshop" (Gunicorn on port 8000)

When a visitor (data packet) arrives at the building, they need both the building address (IP) and the suite number (port) to reach the right office. Sending data to `192.168.1.50:80` is like sending a letter to "123 Main Street, Suite 80".

The building can have up to 65,536 suites (ports 0–65535), though most are empty. The ground floor suites (ports 0–1023) are reserved for major tenants (well-known services), while upper floors are available for anyone.

</details>

### Prompt 2: Why does SSH use port 22?

<details>
<summary>Click to see a sample answer</summary>

SSH uses port 22 by historical convention. When Tatu Ylonen created the SSH protocol in 1995, he chose port 22 because:

1. Port 21 was already taken by FTP (File Transfer Protocol)
2. Port 23 was already taken by Telnet (an older, insecure remote login protocol)
3. Port 22 was available and conveniently located between these two related services

The port number itself has no technical significance — SSH could theoretically run on any port number. Port 22 was simply assigned by the Internet Assigned Numbers Authority (IANA) as the standard port for SSH.

In practice, many system administrators change the SSH port from 22 to a non-standard port (like 2222 or 22222) as a basic security measure. While this does not make SSH more secure, it reduces the number of automated attacks that target the default port.

When you run `ssh user@server`, the SSH client automatically connects to port 22 unless you specify a different port with the `-p` flag: `ssh -p 2222 user@server`.

</details>

---

## 5.5 Activity

### Instructions

1. On **Windows**, open Command Prompt and run:
   ```cmd
   netstat -ano | findstr LISTENING
   ```
   Or on **Ubuntu**, log in and run:
   ```bash
   sudo ss -tuln
   ```

2. Take a screenshot showing the list of open/listening ports

### Submission

- **Upload your screenshot**
- **Caption:** `Open ports listed successfully`

---

## 5.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| `ss` command not found on Ubuntu | Install it with `sudo apt install iproute2` |
| `netstat` not found on Ubuntu | Install it with `sudo apt install net-tools` |
| No ports showing as LISTENING | Make sure you are using `sudo` on Ubuntu. Some ports require elevated privileges to view. |
| Output is overwhelming with too many entries | Use filtering: `sudo ss -tuln` shows only listening ports. On Windows, use `netstat -ano \| findstr LISTENING`. |

---

## 5.7 Summary

In this chapter you learned:

- What network ports are and why they exist
- Common port numbers: 22 (SSH), 80 (HTTP), 443 (HTTPS), 8000 (Gunicorn)
- The difference between well-known, registered, and dynamic port ranges
- How to check open ports on Windows (`netstat -ano`) and Ubuntu (`sudo ss -tuln`)
- How IP addresses and ports combine to form sockets that identify specific services

This knowledge is critical for the next chapter where you will configure firewall rules (allowing or blocking specific ports) and set up SSH remote access (which uses port 22).

**Next:** [Chapter 6 — Firewall, SSH Setup and Remote Login](chapter-06-firewall-ssh-remote-login.md)
