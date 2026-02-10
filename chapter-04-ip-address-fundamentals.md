# Chapter 4 — IP Address Fundamentals

| | |
|---|---|
| **Objective** | Understand IP addresses and verify network connectivity on Windows and Ubuntu |
| **Duration** | 20–30 minutes |
| **Difficulty** | Beginner |
| **Prerequisites** | Chapter 3 completed (comfortable with basic Linux commands) |

---

## 4.1 Overview

Before you can connect to your server remotely, deploy applications, or configure networking, you need to understand **IP addresses** — the fundamental way computers identify and communicate with each other on a network.

In this chapter you will learn what IP addresses are, how they work, and how to check them on both Windows and Ubuntu. This knowledge is essential for the remaining chapters where you will set up SSH connections, configure firewalls, and deploy web applications.

---

## 4.2 Understanding the Concepts

### What Is an IP Address?

An **IP address** (Internet Protocol address) is a unique numerical label assigned to every device connected to a network. It serves two purposes:

1. **Identification** — It tells the network "who" the device is
2. **Location** — It tells the network "where" the device is

**Analogy:** Think of an IP address like a postal address for your home. Just as the postal service uses your street address to deliver mail to you, the internet uses IP addresses to send data to the right computer.

### IPv4 Format

The most common IP address format is **IPv4** (Internet Protocol version 4). It looks like this:

```
192.168.1.100
```

Each IPv4 address consists of **four numbers** separated by dots:

```
192  .  168  .  1  .  100
 │       │      │      │
 │       │      │      └── Host number (0-255)
 │       │      └── Subnet
 │       └── Second octet
 └── First octet

Each number ranges from 0 to 255
```

### Private vs Public IP Addresses

Every device on the internet needs an IP address, but there are not enough IPv4 addresses for every device to have a unique public one. So we use two types:

#### Public IP Address

- Assigned by your **Internet Service Provider (ISP)**
- Unique across the entire internet
- Used for communication between your network and the outside world
- Example: `103.25.178.42`

#### Private IP Address

- Assigned by your **local router**
- Only unique within your local network
- Not accessible from the internet directly
- Used for devices within the same network to communicate with each other

**Private IP address ranges:**

| Range | Class | Commonly Used For |
|-------|-------|-------------------|
| `10.0.0.0` – `10.255.255.255` | Class A | Large organizations, VirtualBox default |
| `172.16.0.0` – `172.31.255.255` | Class B | Medium networks |
| `192.168.0.0` – `192.168.255.255` | Class C | Home and small office networks |

### What Does `192.168.x.x` Mean?

If your IP address starts with `192.168`, it means:

- You are on a **private network** (home, office, or local network)
- This address is **not directly reachable from the internet**
- Your router provides this address to devices on your local network
- Multiple homes can have devices with the same `192.168.x.x` address because these addresses are only meaningful within their own local network

### How Devices Communicate: NAT

```
                    ┌─────────────┐
                    │  Internet   │
                    └──────┬──────┘
                           │
                    Public IP: 103.25.178.42
                           │
                    ┌──────┴──────┐
                    │   Router    │
                    │   (NAT)     │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        192.168.1.10  192.168.1.11  192.168.1.12
        Your PC       Phone         Ubuntu VM
```

Your router uses **NAT** (Network Address Translation) to translate between your private addresses and the single public address. This is how many devices can share one public IP address.

### Special IP Addresses

| Address | Meaning |
|---------|---------|
| `127.0.0.1` | **Localhost** — refers to "this computer itself" |
| `0.0.0.0` | Refers to all network interfaces on the machine |
| `255.255.255.255` | Broadcast address — sends to all devices on the network |
| `10.0.2.15` | Common default IP for VirtualBox VMs using NAT |

### VirtualBox Networking Modes

Your VM's IP address depends on the VirtualBox network mode:

| Mode | VM Gets IP From | Host Can Reach VM? | VM Can Reach Internet? | Typical IP |
|------|----------------|--------------------|-----------------------|------------|
| **NAT** (default) | VirtualBox DHCP | No (without port forwarding) | Yes | `10.0.2.15` |
| **Bridged Adapter** | Your home router | Yes | Yes | `192.168.x.x` |
| **Host-Only** | VirtualBox DHCP | Yes | No | `192.168.56.x` |

> **For this module:** We recommend using **Bridged Adapter** so your host machine (Windows) can directly reach the VM. This makes SSH (Chapter 6) easier to set up. You can change this in VirtualBox: VM Settings > Network > Attached to > Bridged Adapter.

---

## 4.3 Step-by-Step Instructions

### Checking IP Address on Windows

#### Command 1: `ipconfig`

Open **Command Prompt** (press `Windows + R`, type `cmd`, press Enter) and run:

```cmd
ipconfig
```

**Sample output:**
```
Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.1.10
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1
```

**What each line means:**

| Field | Meaning |
|-------|---------|
| **IPv4 Address** | Your computer's IP address on the local network |
| **Subnet Mask** | Defines which part of the IP identifies the network vs the device. `255.255.255.0` means the first three numbers identify the network (`192.168.1`) and the last number identifies the device (`.10`) |
| **Default Gateway** | The IP address of your router — the "exit door" to the internet |

#### Command 2: `ping`

Test connectivity to the internet:

```cmd
ping google.com
```

**Sample output:**
```
Pinging google.com [142.250.190.14] with 32 bytes of data:
Reply from 142.250.190.14: bytes=32 time=12ms TTL=117
Reply from 142.250.190.14: bytes=32 time=11ms TTL=117
Reply from 142.250.190.14: bytes=32 time=13ms TTL=117
Reply from 142.250.190.14: bytes=32 time=12ms TTL=117
```

**What this tells you:**

| Field | Meaning |
|-------|---------|
| `142.250.190.14` | Google's public IP address (your computer resolved the domain name) |
| `bytes=32` | Size of the data packet sent |
| `time=12ms` | Round-trip time — how long it took for data to go to Google and back (lower is better) |
| `TTL=117` | Time to Live — prevents packets from looping forever on the network |

If you see `Reply from...`, your internet connection is working.

If you see `Request timed out`, there may be a connectivity problem.

---

### Checking IP Address on Ubuntu

Open the terminal on your Ubuntu VM (`Ctrl + Alt + T`) and run:

#### Command 1: `ip a`

```bash
ip a
```

**Sample output:**
```
1: lo: <LOOPBACK,UP,LOWER_UP>
    inet 127.0.0.1/8 scope host lo

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.50/24 brd 192.168.1.255 scope global dynamic enp0s3
```

**Reading the output:**

| Part | Meaning |
|------|---------|
| `lo` | **Loopback interface** — the internal "self" address (`127.0.0.1`). Every computer has this. |
| `enp0s3` | **Ethernet interface** — your network card. This is the one that matters. |
| `inet 192.168.1.50/24` | Your VM's IP address is `192.168.1.50`. The `/24` means the subnet mask is `255.255.255.0`. |
| `brd 192.168.1.255` | Broadcast address for your network |
| `dynamic` | The IP was assigned automatically by DHCP (from your router) |

> **Write down your VM's IP address.** You will need it in Chapter 6 to connect via SSH.

#### Command 2: `ping`

```bash
ping -c 4 google.com
```

> **Note:** On Ubuntu, `ping` runs continuously by default. Use `-c 4` to send only 4 packets and stop automatically.

**Sample output:**
```
PING google.com (142.250.190.14) 56(84) bytes of data.
64 bytes from 142.250.190.14: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from 142.250.190.14: icmp_seq=2 ttl=117 time=11.8 ms
64 bytes from 142.250.190.14: icmp_seq=3 ttl=117 time=13.1 ms
64 bytes from 142.250.190.14: icmp_seq=4 ttl=117 time=12.0 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

If you see `0% packet loss`, your VM has internet connectivity.

---

### Verify Host-to-VM Connectivity

From your **Windows** Command Prompt, try pinging your Ubuntu VM:

```cmd
ping 192.168.1.50
```

(Replace `192.168.1.50` with the actual IP of your VM from the `ip a` output.)

If it replies, your Windows machine can communicate with your VM. This is required for SSH in Chapter 6.

If it does not reply, your VM may be using NAT mode. Change the network adapter to **Bridged Adapter** in VirtualBox settings:

1. Power off the VM
2. Go to VM **Settings > Network**
3. Change "Attached to" from **NAT** to **Bridged Adapter**
4. Select your active network adapter from the Name dropdown
5. Click OK and start the VM
6. Run `ip a` again — you should now get a `192.168.x.x` address

---

## 4.4 Self-Study Prompts

### Prompt 1: Explain the difference between a private and a public IP address.

<details>
<summary>Click to see a sample answer</summary>

A **public IP address** is assigned by your Internet Service Provider and is unique across the entire internet. It is the address that external websites and services see when you connect to them. Think of it as the street address of your building — anyone in the world can use it to find you.

A **private IP address** is assigned by your local router and is only meaningful within your own network. Multiple homes can have devices with the same private IP addresses (like `192.168.1.10`) because these addresses are not visible on the internet. Think of it as an apartment number — "Apartment 3B" exists in thousands of buildings, but within your building, it uniquely identifies one unit.

Private IP ranges:
- `10.0.0.0` to `10.255.255.255`
- `172.16.0.0` to `172.31.255.255`
- `192.168.0.0` to `192.168.255.255`

Your router uses NAT (Network Address Translation) to convert between private and public addresses when your devices communicate with the internet.

</details>

### Prompt 2: What does an address like `192.168.x.x` represent?

<details>
<summary>Click to see a sample answer</summary>

An address starting with `192.168` is a **private IP address** belonging to the Class C private range (`192.168.0.0` to `192.168.255.255`). This range is most commonly used in home and small office networks.

When you see `192.168.x.x`:
- The device is on a **local network**, not directly accessible from the internet
- The address was likely assigned by your home router via DHCP
- `192.168.1.x` and `192.168.0.x` are the most common home network ranges
- The first three numbers typically identify the network, and the last number identifies the specific device

For example, in a home network:
- `192.168.1.1` — usually the router itself
- `192.168.1.10` — your computer
- `192.168.1.11` — your phone
- `192.168.1.50` — your Ubuntu VM

These devices can all communicate with each other using these addresses, but none of them are reachable from outside your home network using these addresses.

</details>

---

## 4.5 Activity

### Instructions

1. Open Command Prompt on Windows and run:
   ```cmd
   ipconfig
   ```
2. Open the terminal on your Ubuntu VM (`Ctrl + Alt + T`) and run:
   ```bash
   ip a
   ```
3. Test connectivity from either system:
   ```bash
   ping -c 4 google.com
   ```
4. Take a screenshot showing the IP address output and ping results

### Submission

- **Upload your screenshot**
- **Caption:** `IP address and connectivity verified`

---

## 4.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| `ip a` shows no IP address on the network interface | Your VM may not be connected to a network. Check VirtualBox VM Settings > Network and make sure "Enable Network Adapter" is checked. Try switching to Bridged Adapter mode. |
| `ping google.com` shows "Name or service not known" | DNS is not working. Try `ping 8.8.8.8` instead. If that works, the issue is DNS resolution. Run `sudo systemctl restart systemd-resolved`. |
| `ping google.com` shows "Network is unreachable" | No network connection at all. Verify the VirtualBox network adapter settings. Try restarting the VM. |
| Windows cannot ping the Ubuntu VM | The VM is likely in NAT mode. Switch to Bridged Adapter (see instructions above) or set up port forwarding in VirtualBox. |
| `ipconfig` shows a `169.254.x.x` address | This means Windows could not get an IP from the router (APIPA address). Check your physical network cable or Wi-Fi connection. |

---

## 4.7 Summary

In this chapter you learned:

- What IP addresses are and why they are needed
- The difference between private and public IP addresses
- Common private IP ranges and what `192.168.x.x` means
- How to check your IP address on Windows (`ipconfig`) and Ubuntu (`ip a`)
- How to test connectivity using `ping`
- VirtualBox networking modes and when to use Bridged Adapter

Understanding IP addresses is critical for the next chapters where you will configure firewalls, set up SSH connections, and deploy applications accessible over the network.

**Next:** [Chapter 5 — Ports and Network Communication](chapter-05-ports-and-network-communication.md)
