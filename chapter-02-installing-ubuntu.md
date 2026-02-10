# Chapter 2 — Installing Ubuntu

| | |
|---|---|
| **Objective** | Install Ubuntu inside VirtualBox and verify it boots to the desktop |
| **Duration** | 30–45 minutes |
| **Difficulty** | Beginner |
| **Prerequisites** | Chapter 1 completed (VirtualBox installed) |

---

## 2.1 Overview

Now that VirtualBox is installed, you need an operating system to run inside it. In this chapter, you will download **Ubuntu Desktop** and install it inside a new virtual machine.

Ubuntu is one of the most popular Linux distributions in the world. We are installing the **Desktop edition** so you can get familiar with the Linux graphical interface for general purposes — browsing files, exploring settings, and getting comfortable with a new operating system. However, starting from Chapter 3, our focus shifts to the **terminal and command line**. In real-world server management, you connect remotely via SSH and work entirely through commands. The desktop is your safety net — the terminal is where the real work happens.

> **Why Desktop and not Server?** Ubuntu Desktop gives you a familiar graphical environment to fall back on while you learn. You get the best of both worlds — a GUI for exploration and a terminal for deployment skills. In production, servers run without a desktop, but for learning purposes, having one reduces friction.

By the end of this chapter, you will have a working Ubuntu system that you can log into and use for all the remaining chapters.

---

## 2.2 Understanding the Concepts

### What Is Ubuntu?

Ubuntu is a free, open-source Linux operating system used by millions of people and organizations worldwide. It comes in two main editions:

| Feature | Ubuntu Desktop | Ubuntu Server |
|---------|---------------|---------------|
| **Interface** | Graphical (GUI with windows, mouse, taskbar) | Command line only (terminal) |
| **Resource usage** | Higher (needs more RAM and CPU for graphics) | Lower (no graphical overhead) |
| **Pre-installed software** | Web browser, file manager, text editor, terminal | Minimal tools, SSH server |
| **Use case** | Personal computers, learning, development | Web servers, cloud hosting, production |

For this module we use **Ubuntu Desktop** so you can explore Linux visually, but all deployment work in later chapters will be done through the terminal — just like professionals do on real servers.

### What Is an ISO File?

An ISO file is a **disk image** — a single file that contains the complete contents of an installation disc. Instead of burning it to a physical DVD, VirtualBox can use the ISO file directly as a virtual disc to install the operating system.

### Virtual Machine Resource Allocation

When you create a VM, you assign it a portion of your host machine's resources. Giving the VM enough resources ensures a smooth installation and usable experience:

```
Your Computer (Host): 8 GB RAM, 256 GB Disk
         │
         ├── Host OS (Windows): Uses ~4 GB RAM
         │
         └── Virtual Machine (Ubuntu):
                 Assigned: 4 GB RAM, 2 CPUs, 25 GB Disk
                 (Taken from host's available resources)
```

> **Important:** The resources you assign to the VM are shared from your host. Allocating more RAM and CPUs to the VM makes installation faster and the system more responsive, but leave enough for Windows to run comfortably.

---

## 2.3 Step-by-Step Instructions

### Step 1: Download Ubuntu Desktop ISO

1. Open your web browser
2. Go to: **https://ubuntu.com/download/desktop**
3. Click **Download** for the latest LTS version (LTS stands for Long Term Support — it receives updates for 5 years)
4. The ISO file is approximately **5 GB** — wait for the download to complete
5. Note the location of the downloaded file (usually your `Downloads` folder)

> **Tip:** LTS versions (like 22.04 or 24.04) are recommended because they are stable and supported for a long time. Avoid non-LTS versions for this module.

### Step 2: Create a New Virtual Machine

1. Open **VirtualBox Manager**
2. Click the **New** button (or go to **Machine > New**)
3. Fill in the details:

   | Field | Value |
   |-------|-------|
   | **Name** | `Ubuntu` |
   | **Folder** | Leave as default (or choose a location with enough disk space) |
   | **ISO Image** | Click the dropdown and select **Other**, then browse to the Ubuntu Desktop ISO you downloaded |
   | **Type** | `Linux` |
   | **Version** | `Ubuntu (64-bit)` |

4. Check the box **Skip Unattended Installation** (this lets you go through the installation manually, which is better for learning)
5. Click **Next**

### Step 3: Allocate Hardware Resources

On the **Hardware** screen, give the VM enough resources for a smooth experience:

| Setting | Recommended Value | Minimum Value |
|---------|-------------------|---------------|
| **Base Memory (RAM)** | 4096 MB (4 GB) | 2048 MB (2 GB) |
| **Processors** | 4 CPUs | 2 CPUs |

- Move the **Memory** slider to **4096 MB** (or 2048 MB if your host has only 8 GB RAM)
- Set **Processors** to **4** (or 2 if your host has fewer cores)
- Click **Next**

> **Note:** Allocating more RAM and CPUs significantly speeds up the installation process and makes the desktop responsive. If your host machine has 16 GB RAM, feel free to give the VM 4 GB. If your host has 8 GB, use 2 GB for the VM.

### Step 4: Create Virtual Hard Disk

On the **Virtual Hard Disk** screen:

1. Select **Create a Virtual Hard Disk Now**
2. Set the **Disk Size** to **25 GB** (sufficient for our exercises plus some room for packages)
3. Leave the option **Pre-allocate Full Size** unchecked (this means VirtualBox will only use disk space as the VM needs it, not all 25 GB immediately)
4. Click **Next**

### Step 5: Review and Create

1. Review the summary of your VM settings
2. Click **Finish**
3. The new VM will appear in the VirtualBox Manager list on the left

### Step 6: Start the VM and Begin Installation

1. Select your **Ubuntu** VM in the list
2. Click the **Start** button (green arrow)
3. A new window will open — this is your virtual machine's screen
4. The Ubuntu installer will boot from the ISO (this may take a minute)

### Step 7: Ubuntu Installation Process

Follow these steps in the Ubuntu graphical installer:

1. **Welcome Screen:**
   - Select your language (e.g., **English**)
   - Click **Install Ubuntu**

2. **Keyboard Layout:**
   - Select **English (US)** (or your preferred layout)
   - Click **Continue**

3. **Updates and Other Software:**
   - Select **Normal installation** (includes web browser, utilities, and office tools)
   - Check **Download updates while installing Ubuntu** (if you have internet)
   - Click **Continue**

4. **Installation Type:**
   - Select **Erase disk and install Ubuntu**
   - Click **Install Now**
   - A confirmation dialog will appear — click **Continue** (this only erases the virtual disk, not your real hard drive)

5. **Where Are You? (Timezone):**
   - Select your timezone by clicking on the map or typing your city
   - Click **Continue**

6. **Who Are You? (User Setup):**

   | Field | What to Enter |
   |-------|---------------|
   | **Your name** | Your full name (e.g., `Student`) |
   | **Your computer's name** | `ubuntu-vm` |
   | **Pick a username** | `ubuntu` (the default convention for Ubuntu systems) |
   | **Password** | Choose a password you will remember |
   | **Confirm password** | Same password again |
   | **Log in automatically** | Leave unchecked (use password login for security practice) |

   > **Write down your username and password.** You will need them every time you log in and for `sudo` commands.

   - Click **Continue**

7. **Installation Progress:**
   - Wait for the installation to complete (this takes 10–20 minutes depending on your internet speed and VM resources)
   - When you see **Installation Complete**, click **Restart Now**

8. **After Reboot:**
   - If you see a message saying "Please remove the installation medium, then press ENTER", just press **Enter**
   - Wait for Ubuntu to boot to the login screen
   - Click your username and enter your password

### Step 8: You Are In — The Ubuntu Desktop

After logging in, you will see the Ubuntu desktop with:
- A **dock** on the left side (similar to the Windows taskbar)
- A **Files** app for browsing files
- A **Firefox** browser
- A **top bar** showing the time, network, and system settings

Take a moment to explore the desktop. Click around, open the file manager, check the settings. This is your Linux environment.

### Step 9: Open the Terminal

The terminal is where you will spend most of your time in this module. Open it now:

1. Press **Ctrl + Alt + T** (keyboard shortcut to open the terminal)
   - Or click the **Activities** button (top-left), type `Terminal`, and click it

2. You should see a window with a prompt like:
   ```
   ubuntu@ubuntu-vm:~$
   ```

3. Run the following command to verify your installation:
   ```bash
   uname -a
   ```

4. You should see output similar to:
   ```
   Linux ubuntu-vm 6.x.0-xx-generic #xx-Ubuntu SMP ... x86_64 GNU/Linux
   ```

   This confirms:
   - `Linux` — the kernel
   - `ubuntu-vm` — your hostname
   - `x86_64` — 64-bit architecture
   - `GNU/Linux` — the operating system

---

## 2.4 Self-Study Prompts

### Prompt 1: What are the steps to install Ubuntu in VirtualBox?

<details>
<summary>Click to see a sample answer</summary>

1. Download the Ubuntu Desktop ISO file from ubuntu.com
2. Open VirtualBox and click "New" to create a virtual machine
3. Name the VM, select the ISO image, and choose Linux/Ubuntu as the type
4. Allocate RAM (2–4 GB) and CPUs (2–4 cores) for a smooth experience
5. Create a virtual hard disk (25 GB)
6. Start the VM — it boots from the ISO into the graphical installer
7. Choose language, keyboard, timezone, and select "Normal installation"
8. Select "Erase disk and install Ubuntu" (this only affects the virtual disk)
9. Create a user account with a username and password
10. Wait for installation, reboot, and log in to the Ubuntu desktop

</details>

### Prompt 2: Why are we installing Ubuntu Desktop instead of Ubuntu Server for this module?

<details>
<summary>Click to see a sample answer</summary>

We install Ubuntu Desktop for several practical reasons:

1. **Lower barrier to entry** — A graphical desktop is familiar to students coming from Windows. You can browse files visually, use a web browser, and explore settings without needing to know commands yet.

2. **Safety net** — If you get stuck with commands, you can fall back to the GUI to fix things (edit files with a text editor, check network settings visually, etc.).

3. **General Linux familiarity** — Getting comfortable with the Ubuntu desktop environment is valuable even outside of server work. Many developers use Linux as their daily operating system.

However, our module's focus is on **terminal and SSH commands** — this is how real servers are managed. In production environments (AWS, Azure, DigitalOcean), servers have no graphical interface at all. Starting from Chapter 3, we will work primarily in the terminal, and from Chapter 6, we will connect via SSH — exactly as you would with a real remote server.

The desktop is for comfort. The terminal is where the skills are built.

</details>

---

## 2.5 Activity

### Instructions

1. Complete the Ubuntu installation inside VirtualBox
2. Log in to the Ubuntu desktop
3. Open the terminal (`Ctrl + Alt + T`)
4. Run the following command:
   ```bash
   uname -a
   ```
5. Take a screenshot showing the terminal output on the Ubuntu desktop

### Submission

- **Upload your screenshot**
- **Caption:** `Ubuntu installed successfully`

---

## 2.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| VM boots to a black screen and nothing happens | Wait 1–2 minutes. If still blank, power off the VM (Machine > Power Off) and try again. Make sure the ISO file path is correct. |
| "This kernel requires an x86-64 CPU" error | Make sure you selected **Ubuntu (64-bit)** when creating the VM, not 32-bit. Also ensure hardware virtualization is enabled in BIOS (see Chapter 1 troubleshooting). |
| Desktop is extremely slow and laggy | Increase allocated RAM to 4 GB and processors to 4 in VM Settings > System. Also try: VM Settings > Display > increase Video Memory to 128 MB. |
| Screen resolution is too small | Inside Ubuntu, go to Settings > Displays and change the resolution. Or install Guest Additions: in the VirtualBox menu, click Devices > Insert Guest Additions CD Image, then run the installer inside Ubuntu. |
| No internet connection after installation | Check VirtualBox VM Settings > Network. Make sure "Enable Network Adapter" is checked and "Attached to" is set to NAT or Bridged Adapter. |
| Installation seems frozen at "Downloading updates" | This depends on your internet speed. You can skip updates during installation (uncheck the option) and update later with `sudo apt update && sudo apt upgrade`. |
| "Cannot mount ISO" or no boot device found | Go to VM Settings > Storage, click the empty disc icon under Controller: IDE, click the disc icon on the right, and choose the Ubuntu ISO file. |

---

## 2.7 Summary

In this chapter you:

- Downloaded the Ubuntu Desktop ISO
- Created a new virtual machine in VirtualBox with 2–4 GB RAM, 2–4 CPUs, and 25 GB disk
- Completed the Ubuntu graphical installation process
- Logged in to the desktop and explored the environment
- Opened the terminal and verified the installation with `uname -a`

You now have a working Linux environment with both a graphical desktop and a terminal. The desktop is there for you to explore and get comfortable with Linux. From the next chapter onward, we shift our focus to the terminal — the tool that powers real server administration.

**Next:** [Chapter 3 — Basic Ubuntu Commands](chapter-03-basic-ubuntu-commands.md)
