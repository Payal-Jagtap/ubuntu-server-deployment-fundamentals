# Chapter 1 — Installing VirtualBox

| | |
|---|---|
| **Objective** | Install Oracle VirtualBox and verify it launches correctly |
| **Duration** | 15–20 minutes |
| **Difficulty** | Beginner |
| **Prerequisites** | A computer running Windows 10/11 with at least 8 GB RAM |

---

## 1.1 Overview

Before you can set up a Linux environment, you need a place to run it. Most students do not have a spare physical computer, so we use **virtualization** — a technology that lets you run an entire operating system inside a window on your existing computer.

In this chapter, you will install **Oracle VirtualBox**, a free and open-source virtualization application. Think of VirtualBox as a container that holds a virtual computer. Once installed, you will use it throughout this module to run Ubuntu.

---

## 1.2 Step-by-Step Instructions

### Step 1: Download VirtualBox

1. Open your web browser
2. Go to: **https://www.virtualbox.org/wiki/Downloads**
3. Under **VirtualBox platform packages**, click **Windows hosts**
4. The download will start automatically (the file is approximately 100 MB)
5. Wait for the download to complete

### Step 2: Install VirtualBox

1. Locate the downloaded file (usually in your `Downloads` folder). It will be named something like `VirtualBox-7.x.x-xxxxx-Win.exe`
2. **Double-click** the file to start the installer
3. If Windows asks "Do you want to allow this app to make changes to your device?" — click **Yes**
4. The setup wizard will appear. Click **Next**
5. On the **Custom Setup** screen, leave all default options selected. Click **Next**
6. On the **Warning: Network Interfaces** screen, it will inform you that your network will be temporarily disconnected during installation. Click **Yes**
7. On the **Missing Dependencies** screen (if shown), click **Yes** to install any required packages
8. Click **Install** to begin installation
9. Wait for the installation to complete (this takes 1–3 minutes)
10. Click **Finish** to close the installer (leave "Start Oracle VirtualBox" checked)

### Step 3: Verify Installation

1. VirtualBox should open automatically after installation. If not, find it in your Start Menu and open it.
2. You should see the **VirtualBox Manager** window — this is the main interface where you will create and manage virtual machines.
3. The window should show an empty list (you have not created any VMs yet) with options like **New**, **Add**, **Settings**, etc.

Your screen should look similar to this:

```
┌────────────────────────────────────────────────────────────┐
│  Oracle VirtualBox Manager                            _ □ X│
├────────────────────────────────────────────────────────────┤
│  File  Machine  Help                                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Tools ▼                                                   │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                                                      │  │
│  │           Welcome to VirtualBox!                     │  │
│  │                                                      │  │
│  │     [ New ]  [ Add ]  [ Settings ]  [ Import ]       │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 1.3 Understanding the Concepts

Now that VirtualBox is installed, let's understand what it actually does.

### What Is Virtualization?

Virtualization is the process of creating a **virtual (software-based) version of a computer** inside your physical computer. The virtual computer has its own CPU, RAM, hard disk, and network interface — but these are all **simulated by software** using your real computer's resources.

**Analogy:** Think of your physical computer as a large house. Virtualization lets you build smaller apartments inside that house. Each apartment (virtual machine) has its own rooms, kitchen, and bathroom, but they all share the same building structure (your physical hardware).

### Key Terminology

| Term | Definition |
|------|------------|
| **Host Machine** | Your physical computer (the one you are sitting at right now) |
| **Guest Machine** | The virtual computer running inside VirtualBox |
| **Virtual Machine (VM)** | Another name for the guest machine |
| **Hypervisor** | The software that creates and manages virtual machines. VirtualBox is a Type 2 hypervisor (it runs on top of your existing OS). |
| **ISO File** | A disk image file used to install an operating system (like a virtual DVD) |

### Why Do Developers Use Virtual Machines?

1. **Safety** — If you break something inside a VM, your real computer is unaffected. You can delete the VM and start over.
2. **Isolation** — Each VM is separated from your host machine. Software installed in a VM does not interfere with your host.
3. **Portability** — VMs can be exported, shared, and imported on different computers.
4. **Cost** — Instead of buying multiple physical servers, you can run several VMs on one machine.
5. **Learning** — You can practice server administration, networking, and deployment without risking your personal computer.

### Types of Hypervisors

```
Type 1 (Bare Metal)                    Type 2 (Hosted)
┌──────────────────────┐              ┌──────────────────────┐
│   Virtual Machine    │              │   Virtual Machine    │
├──────────────────────┤              ├──────────────────────┤
│   Hypervisor         │              │   VirtualBox         │
├──────────────────────┤              ├──────────────────────┤
│   Physical Hardware  │              │   Host OS (Windows)  │
└──────────────────────┘              ├──────────────────────┤
                                      │   Physical Hardware  │
                                      └──────────────────────┘
Examples: VMware ESXi, Hyper-V        Examples: VirtualBox, VMware Workstation
Used in: Data centers, cloud          Used in: Learning, development, testing
```

VirtualBox is a **Type 2 hypervisor** — it installs as a regular application on your Windows/Mac/Linux computer and runs VMs as windows on your desktop.

---

## 1.4 Self-Study Prompts

Answer these questions in your own words. Writing out explanations helps reinforce your understanding.

### Prompt 1: Explain virtualization in simple terms.

<details>
<summary>Click to see a sample answer</summary>

Virtualization is the process of using software to create a simulated computer inside your real computer. The simulated computer (called a virtual machine) behaves like a real, separate computer — it has its own operating system, its own files, and its own network connection — but it actually shares the physical hardware (CPU, RAM, disk) of your real computer.

Think of it like running a computer inside a window on your desktop. You can start it, stop it, and delete it without affecting your actual computer.

</details>

### Prompt 2: Why do developers use virtual machines instead of installing operating systems directly?

<details>
<summary>Click to see a sample answer</summary>

Developers use virtual machines for several reasons:

1. **No risk to the main system** — If a VM gets corrupted or misconfigured, you simply delete it and create a new one. Your host computer stays safe.
2. **Multiple environments** — You can run different operating systems (Windows, Linux, macOS) on the same physical machine simultaneously.
3. **Easy to replicate** — VMs can be cloned or exported, making it easy to share identical environments across a team.
4. **Snapshots** — You can save the state of a VM at any point and revert back to it if something goes wrong.
5. **Cost savings** — Running multiple VMs on one physical machine is cheaper than buying multiple physical servers.
6. **Testing** — You can test software in different environments without needing separate hardware for each.

</details>

---

## 1.5 Activity

### Instructions

1. Complete the installation steps described above
2. Launch VirtualBox
3. Take a screenshot showing the VirtualBox Manager home screen

### Submission

- **Upload your screenshot**
- **Caption:** `VirtualBox installed successfully`

---

## 1.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| Installer says "VT-x is not available" | Hardware virtualization needs to be enabled in your BIOS. Restart your computer, press the BIOS key during startup (usually `F2`, `F10`, `Del`, or `Esc`), find a setting called **Intel VT-x**, **AMD-V**, or **Virtualization Technology**, set it to **Enabled**, save and exit. |
| Installer fails with permissions error | Right-click the installer and select **Run as administrator** |
| VirtualBox opens but shows warnings about kernel drivers | Reinstall VirtualBox, making sure to click **Yes** when asked about installing device drivers |
| Download is very slow | Try using a different browser or a wired internet connection |
| Windows Defender blocks the installer | Temporarily disable real-time protection in Windows Security settings, install VirtualBox, then re-enable it |

> **Note on Hardware Virtualization:** Most modern computers have virtualization enabled by default. You only need to touch BIOS settings if VirtualBox gives you an error. Do not worry about this unless you see a problem.

---

## 1.7 Summary

In this chapter you:

- Downloaded and installed Oracle VirtualBox
- Verified that VirtualBox launches correctly
- Learned what virtualization is, why it is used, and how VirtualBox fits in

**Next:** [Chapter 2 — Installing Ubuntu](chapter-02-installing-ubuntu.md)
