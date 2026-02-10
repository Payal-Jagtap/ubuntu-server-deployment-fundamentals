# Chapter 7 — Deploying Flask Application from Git

| | |
|---|---|
| **Objective** | Clone a Flask application from Git and run it using Gunicorn |
| **Duration** | 40–50 minutes |
| **Difficulty** | Intermediate |
| **Prerequisites** | Chapter 6 completed (SSH access to Ubuntu VM working) |

---

## 7.1 Overview

In previous chapters, you set up the server infrastructure — the operating system, networking, firewall, and remote access. Now it is time to actually **deploy an application**.

In this chapter you will:
1. Install Git, Python, and related tools on the server
2. Clone a Flask web application from a Git repository
3. Set up a Python virtual environment
4. Install the application's dependencies
5. Run the application using Gunicorn, a production-grade WSGI server

By the end of this chapter, your Flask application will be running on the server and responding to web requests.

---

## 7.2 Understanding the Concepts

### What Is Git-Based Deployment?

In professional software development, code is stored in a **Git repository** (usually on GitHub, GitLab, or Bitbucket). When you deploy an application to a server, you **clone** (download) the repository onto the server instead of manually copying files.

```
Developer's Computer                      Server
┌──────────────────┐                     ┌──────────────────┐
│                  │     git push        │                  │
│  Write Code ─────┼────────────────►    │                  │
│                  │                     │   GitHub/GitLab   │
└──────────────────┘                     │                  │
                                         └────────┬─────────┘
                                                  │ git clone
                                                  ▼
                                         ┌──────────────────┐
                                         │                  │
                                         │  Ubuntu VM       │
                                         │  (your VM)       │
                                         │                  │
                                         └──────────────────┘
```

**Advantages of Git-based deployment:**
- Code is versioned — you can roll back to any previous version
- Deployment is repeatable — clone the same repo on any server
- Changes are tracked — you can see who changed what and when
- Easy updates — just `git pull` to get the latest code

### What Is a Python Virtual Environment?

A **virtual environment** (venv) is an isolated Python environment that keeps your project's dependencies separate from the system's Python packages.

**Why is this important?**

Imagine you have two Python projects on the same server:
- Project A needs Flask version 2.0
- Project B needs Flask version 3.0

If both projects share the same Python installation, you cannot have two different versions of Flask installed at the same time. A virtual environment solves this by giving each project its own isolated set of packages.

```
Server's Python
├── System packages (used by Ubuntu itself — DO NOT modify)
│
├── Project A Virtual Environment
│   └── Flask 2.0, requests 2.28, ...
│
└── Project B Virtual Environment
    └── Flask 3.0, requests 2.31, ...
```

**Rules:**
- Always use a virtual environment for Python projects on servers
- Never install project packages into the system Python using `sudo pip install`
- Each project gets its own `venv` directory

### What Is Gunicorn?

**Gunicorn** (Green Unicorn) is a **WSGI HTTP server** for Python web applications. Let's break that down:

- **WSGI** (Web Server Gateway Interface) — a standard that defines how Python web applications (like Flask) communicate with web servers
- **HTTP Server** — a program that listens for web requests and sends back responses

**Why not just use Flask's built-in server?**

Flask comes with a development server (`flask run`), but it has serious limitations:

| Feature | Flask Dev Server | Gunicorn |
|---------|-----------------|----------|
| **Concurrent requests** | Handles one at a time | Handles many simultaneously (multiple workers) |
| **Performance** | Slow, not optimized | Fast, production-grade |
| **Stability** | Can crash under load | Designed for 24/7 operation |
| **Security** | Not hardened | Production-ready |
| **Recommended for** | Development/testing only | Production deployment |

Flask's own documentation warns:

> "Do not use the development server when deploying to production. It is intended for use only during local development."

**How Gunicorn works:**

```
Incoming HTTP Request
        │
        ▼
   ┌──────────┐
   │ Gunicorn  │  (Master Process)
   │           │
   │  ┌─────┐ │
   │  │ W1  │ │  Worker 1 ──► Flask App ──► Response
   │  └─────┘ │
   │  ┌─────┐ │
   │  │ W2  │ │  Worker 2 ──► Flask App ──► Response
   │  └─────┘ │
   │  ┌─────┐ │
   │  │ W3  │ │  Worker 3 ──► Flask App ──► Response
   │  └─────┘ │
   └──────────┘

Gunicorn spawns multiple worker processes.
Each worker can handle requests independently.
This allows multiple users to be served simultaneously.
```

---

## 7.3 Step-by-Step Instructions

Connect to your Ubuntu VM via SSH from your Windows terminal:

```cmd
ssh ubuntu@192.168.1.50
```

(Replace with your actual username and IP.)

### Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

This ensures all packages are up to date before installing new software.

### Step 2: Install Required Packages

```bash
sudo apt install git python3 python3-pip python3-venv -y
```

**What each package does:**

| Package | Purpose |
|---------|---------|
| `git` | Version control — used to clone the application repository |
| `python3` | The Python interpreter (usually already installed on Ubuntu) |
| `python3-pip` | Python package manager — installs Python libraries |
| `python3-venv` | Tool for creating Python virtual environments |

### Step 3: Verify Installations

```bash
git --version
python3 --version
pip3 --version
```

You should see version numbers for each tool, confirming they are installed.

### Step 4: Clone the Application Repository

Navigate to a suitable directory and clone the repository:

```bash
cd ~
git clone <repo_url>
```

> **Replace `<repo_url>`** with the actual URL of your Flask application repository. For example:
> ```bash
> git clone https://github.com/yourusername/your-flask-app.git
> ```

After cloning, navigate into the project directory:

```bash
cd your-flask-app
```

> **Replace `your-flask-app`** with the actual name of the cloned directory.

### Step 5: Examine the Project Structure

Take a look at what is in the repository:

```bash
ls -la
```

A typical Flask project looks like:

```
your-flask-app/
├── app.py              ← Main Flask application file
├── requirements.txt    ← List of Python dependencies
├── templates/          ← HTML template files
├── static/             ← CSS, JavaScript, images
└── README.md           ← Project documentation
```

Look at the main application file to understand its structure:

```bash
cat app.py
```

A basic Flask app looks like:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, World!"

if __name__ == '__main__':
    app.run()
```

The key line for Gunicorn is `app = Flask(__name__)` — Gunicorn needs to know the name of the Flask application object (in this case, `app` in the file `app.py`).

### Step 6: Create a Python Virtual Environment

```bash
python3 -m venv venv
```

**What this does:**
- `python3 -m venv` — run the venv module
- `venv` — the name of the directory to create (convention is to name it `venv`)

This creates a `venv/` directory inside your project containing an isolated Python installation.

### Step 7: Activate the Virtual Environment

```bash
source venv/bin/activate
```

After activation, your prompt will change:

```
(venv) ubuntu@ubuntu-vm:~/your-flask-app$
```

The `(venv)` prefix tells you the virtual environment is active. Any Python packages you install now will go into this isolated environment, not the system Python.

**Important:** You must activate the virtual environment every time you open a new terminal session before working with this project.

### Step 8: Install Dependencies

If the project has a `requirements.txt` file:

```bash
pip install -r requirements.txt
```

If there is no `requirements.txt`, install the minimum required packages:

```bash
pip install flask gunicorn
```

Verify the installations:

```bash
pip list
```

You should see Flask, Gunicorn, and their dependencies listed.

### Step 9: Test the Flask Application

Before using Gunicorn, verify the app works with Flask's built-in server:

```bash
python app.py
```

Or:

```bash
flask run --host=0.0.0.0
```

> **Note:** `--host=0.0.0.0` makes the server listen on all network interfaces, not just localhost. This is required for the app to be accessible from outside the VM.

If it starts without errors, press `Ctrl + C` to stop it.

### Step 10: Run with Gunicorn

Now start the application using Gunicorn:

```bash
gunicorn --bind 0.0.0.0:8000 app:app
```

**Breaking down the command:**

| Part | Meaning |
|------|---------|
| `gunicorn` | The Gunicorn WSGI server |
| `--bind 0.0.0.0:8000` | Listen on all network interfaces, port 8000 |
| `app:app` | `filename:variable` — look in `app.py` for the variable named `app` (the Flask application object) |

**Expected output:**
```
[INFO] Starting gunicorn 21.2.0
[INFO] Listening at: http://0.0.0.0:8000
[INFO] Using worker: sync
[INFO] Booting worker with pid: 12345
```

### Step 11: Test the Application

Open a **new** terminal window on Windows (keep Gunicorn running in the first one) and test:

**Option A: Using curl from another SSH session**

```bash
ssh ubuntu@192.168.1.50
curl http://localhost:8000
```

**Option B: Using a browser on Windows**

Open your browser and go to:
```
http://192.168.1.50:8000
```

(Replace with your VM's IP address.)

You should see the response from your Flask application (e.g., "Hello, World!").

### Step 12: Stop Gunicorn

In the terminal where Gunicorn is running, press `Ctrl + C` to stop it.

---

### Running Gunicorn with Multiple Workers

For better performance, you can run Gunicorn with multiple worker processes:

```bash
gunicorn --bind 0.0.0.0:8000 --workers 3 app:app
```

**How many workers?** The recommended formula is:

```
workers = (2 × CPU cores) + 1
```

Since your VM has 2 CPU cores, 5 workers would be ideal, but 3 is fine for learning.

---

### Running Gunicorn in the Background

To keep Gunicorn running after you close your SSH session, you can use `nohup`:

```bash
nohup gunicorn --bind 0.0.0.0:8000 --workers 3 app:app &
```

- `nohup` — prevents the process from stopping when you disconnect
- `&` — runs the process in the background

To stop it later:

```bash
pkill gunicorn
```

> **Note:** In Chapter 8, Nginx will be configured as a reverse proxy in front of Gunicorn. In a production setup, you would also create a **systemd service** to manage Gunicorn automatically (start on boot, restart on crash).

---

## 7.4 Self-Study Prompts

### Prompt 1: What is Gunicorn and why is it used instead of Flask's built-in server?

<details>
<summary>Click to see a sample answer</summary>

**Gunicorn** (Green Unicorn) is a production-grade WSGI HTTP server for Python web applications. WSGI (Web Server Gateway Interface) is a standard that defines how Python web apps communicate with web servers.

Flask includes a built-in development server, but it should **never** be used in production because:

1. **Single-threaded:** Flask's dev server handles one request at a time. If two users visit your site simultaneously, one has to wait for the other to finish.

2. **Not optimized:** The dev server prioritizes ease of debugging over performance. It reloads code on changes and provides debug information — features that are useful during development but wasteful in production.

3. **Security:** The dev server is not hardened against attacks. It may expose debug information that attackers can exploit.

Gunicorn solves all these problems:
- It spawns multiple **worker processes**, each capable of handling requests independently
- It is designed for high performance and 24/7 reliability
- It properly handles signals, graceful shutdowns, and worker management
- It is battle-tested in production by major companies

**Example:** With 3 Gunicorn workers, your app can handle 3 simultaneous requests. With Flask's dev server, it can only handle 1.

</details>

### Prompt 2: Why are virtual environments required?

<details>
<summary>Click to see a sample answer</summary>

Virtual environments are required for several important reasons:

1. **Dependency isolation:** Different projects may need different versions of the same package. Project A might need Flask 2.0 while Project B needs Flask 3.0. Without virtual environments, you can only have one version installed at a time.

2. **System protection:** Ubuntu uses Python for many system tools. If you install or update packages globally with `sudo pip install`, you might break system utilities that depend on specific package versions. A virtual environment keeps your project packages separate from system packages.

3. **Reproducibility:** With a virtual environment and a `requirements.txt` file, you can recreate the exact same set of packages on any machine. This ensures your app works the same way on your server as it does on your development machine.

4. **Clean uninstallation:** To remove all packages for a project, you just delete the `venv` directory. No need to track down and uninstall packages individually.

5. **No sudo required:** Installing packages into a virtual environment does not require root privileges, which is a security best practice.

**Workflow:**
```bash
python3 -m venv venv          # Create the environment
source venv/bin/activate      # Activate it
pip install flask gunicorn    # Install packages (into venv only)
deactivate                    # Deactivate when done
```

</details>

---

## 7.5 Activity

### Instructions

1. SSH into your Ubuntu VM
2. Install Git, Python, pip, and venv
3. Clone your Flask application repository
4. Create and activate a virtual environment
5. Install Flask and Gunicorn
6. Run the application with Gunicorn:
   ```bash
   gunicorn --bind 0.0.0.0:8000 app:app
   ```
7. Test by accessing `http://<your_vm_ip>:8000` in your browser
8. Take a screenshot showing the Gunicorn output in the terminal (or the app in the browser)

### Submission

- **Upload your screenshot**
- **Caption:** `Flask application running`

---

## 7.6 Troubleshooting

| Problem | Solution |
|---------|----------|
| `git clone` fails with "Repository not found" | Double-check the repository URL. If it is a private repo, you need to authenticate. |
| `python3: command not found` | Run `sudo apt install python3` |
| `pip install` gives "externally-managed-environment" error | You are trying to install outside a virtual environment. Make sure you activated the venv first: `source venv/bin/activate` |
| `gunicorn: command not found` | Make sure the virtual environment is activated. If it is, reinstall: `pip install gunicorn` |
| `ModuleNotFoundError: No module named 'flask'` | Activate the virtual environment and install Flask: `source venv/bin/activate && pip install flask` |
| Gunicorn starts but browser shows "Connection refused" | Make sure you used `--bind 0.0.0.0:8000` (not `127.0.0.1:8000`). Also check the firewall: `sudo ufw allow 8000` |
| `app:app` gives "Failed to find application" | The first `app` is the filename (without .py), the second `app` is the Flask variable name. Check your file: if the Flask app variable is `application`, use `app:application`. |
| Application works on VM but not from Windows browser | Check: (1) VM is in Bridged mode, (2) `--bind 0.0.0.0`, (3) firewall allows port 8000 |

---

## 7.7 Summary

In this chapter you:

- Installed Git, Python, pip, and venv on Ubuntu
- Cloned a Flask application from a Git repository
- Created and activated a Python virtual environment
- Installed Flask and Gunicorn as project dependencies
- Ran the Flask application with Gunicorn on port 8000
- Verified the application responds to HTTP requests

The application is now running, but it is served directly by Gunicorn on port 8000. In the next chapter, you will set up **Nginx** as a reverse proxy to:
- Serve the application on the standard web port (80)
- Handle static files efficiently
- Add a layer of security and performance

**Next:** [Chapter 8 — Configuring Nginx for Flask Deployment](chapter-08-configuring-nginx.md)
