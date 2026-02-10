# Module: Ubuntu Server Deployment Fundamentals

## About This Module

This module is a hands-on, self-paced guide that takes you from zero to deploying a live Flask web application on Ubuntu running inside a virtual machine. By the end, you will have built a complete deployment pipeline — from installing the operating system to serving your application through a production-grade web server.

**No prior Linux or server experience is required.** Each chapter builds on the previous one, so follow them in order.

---

## Who Is This For?

- Students learning web development and wanting to understand how applications are deployed
- Developers who have only worked locally and want to learn server administration
- Anyone preparing for DevOps, cloud computing, or system administration roles

---

## What You Will Build

By completing all 10 chapters, you will have:

1. A fully working Ubuntu system running inside VirtualBox
2. SSH remote access configured from your host machine
3. A Python Flask application deployed from a Git repository
4. Gunicorn serving your application as a production WSGI server
5. Nginx acting as a reverse proxy to handle incoming web traffic
6. Proper firewall rules, user management, and file permissions in place

---

## Prerequisites

- A computer running **Windows 10/11** (macOS and Linux users can follow along with minor adjustments)
- At least **4 GB of RAM** (8 GB recommended)
- At least **25 GB of free disk space**
- A stable internet connection for downloading software and packages

---

## Module Outline

| Chapter | Title | What You Will Learn |
|---------|-------|---------------------|
| [Chapter 1](chapters/chapter-01-installing-virtualbox.md) | Installing VirtualBox | Virtualization concepts, installing VirtualBox |
| [Chapter 2](chapters/chapter-02-installing-ubuntu.md) | Installing Ubuntu | Creating a VM, installing Ubuntu OS |
| [Chapter 3](chapters/chapter-03-basic-ubuntu-commands.md) | Basic Ubuntu Commands | Terminal navigation, file and folder management |
| [Chapter 4](chapters/chapter-04-ip-address-fundamentals.md) | IP Address Fundamentals | IP addresses, private vs public, connectivity testing |
| [Chapter 5](chapters/chapter-05-ports-and-network-communication.md) | Ports and Network Communication | Network ports, common port numbers, listing open ports |
| [Chapter 6](chapters/chapter-06-firewall-ssh-remote-login.md) | Firewall, SSH Setup and Remote Login | Firewall rules, SSH installation, remote access |
| [Chapter 7](chapters/chapter-07-deploying-flask-from-git.md) | Deploying Flask Application from Git | Git clone, virtual environments, Gunicorn |
| [Chapter 8](chapters/chapter-08-configuring-nginx.md) | Configuring Nginx for Flask Deployment | Nginx installation, reverse proxy configuration |
| [Chapter 9](chapters/chapter-09-user-management.md) | User Management | Linux users, groups, sudo, creating accounts |
| [Chapter 10](chapters/chapter-10-file-permissions.md) | File Permissions | File ownership, chmod, chown, securing files |

---

## How to Use This Module

1. **Read the overview** at the start of each chapter to understand the goal.
2. **Follow the step-by-step instructions** — type every command yourself instead of copy-pasting. This builds muscle memory.
3. **Read the "Understanding the Concepts" section** to deepen your knowledge beyond just running commands.
4. **Answer the self-study prompts** in your own words before looking at the provided explanations.
5. **Complete the activity** at the end of each chapter and take a screenshot as evidence of your work.
6. **Review the troubleshooting section** if you get stuck — common issues and their fixes are documented.

---

## Architecture Overview

Here is how all the pieces connect once you complete the module:

```
┌─────────────────────────────────────────────────────────────┐
│  YOUR COMPUTER (Host Machine - Windows)                     │
│                                                             │
│   Browser ──────┐                                           │
│                 │                                           │
│   ┌─────────────▼─────────────────────────────────────────┐ │
│   │  VIRTUALBOX (Virtual Machine - Ubuntu)                 │ │
│   │                                                       │ │
│   │   Port 80 ──► Nginx (Reverse Proxy)                   │ │
│   │                   │                                   │ │
│   │                   ▼                                   │ │
│   │              Gunicorn (WSGI Server, Port 8000)        │ │
│   │                   │                                   │ │
│   │                   ▼                                   │ │
│   │              Flask Application (Python)               │ │
│   │                   │                                   │ │
│   │                   ▼                                   │ │
│   │              Git Repository (Source Code)              │ │
│   │                                                       │ │
│   │   SSH (Port 22) ◄── Your Terminal                     │ │
│   │   UFW Firewall: Ports 22, 80 allowed                  │ │
│   └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Tips for Success

- **Type commands manually** — do not copy-paste. You learn by doing.
- **Read error messages carefully** — they usually tell you exactly what went wrong.
- **If something breaks, do not panic** — virtual machines can be reset or rebuilt easily.
- **Use the self-study prompts** — explaining concepts in your own words is the best way to learn.
- **Ask for help when stuck** — but first try troubleshooting on your own for at least 10 minutes.

---

## Let's Begin

Start with **[Chapter 1 — Installing VirtualBox](chapters/chapter-01-installing-virtualbox.md)** and work your way through each chapter in order.

Good luck!
