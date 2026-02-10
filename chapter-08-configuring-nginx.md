# Chapter 8 — Configuring Nginx for Flask Deployment

| | |
|---|---|
| **Objective** | Configure Nginx as a reverse proxy to serve the Flask application on port 80 |
| **Duration** | 30–40 minutes |
| **Difficulty** | Intermediate |
| **Prerequisites** | Chapter 7 completed (Flask app running with Gunicorn) |

---

## 8.1 Overview

In the previous chapter, your Flask application was running with Gunicorn on port 8000. While this works, there are problems:

- Users have to type `http://192.168.1.50:8000` — the `:8000` is ugly and hard to remember
- Gunicorn is not designed to be directly exposed to the internet
- There is no efficient handling of static files (CSS, JavaScript, images)
- There is no protection against slow clients or malicious requests

The solution is **Nginx** — a high-performance web server that sits in front of Gunicorn and handles all incoming web traffic. Nginx acts as a **reverse proxy**, forwarding requests to Gunicorn and sending responses back to the user.

After this chapter, users will be able to access your application by simply visiting `http://192.168.1.50` (no port number needed).

---

## 8.2 Understanding the Concepts

### What Is Nginx?

**Nginx** (pronounced "engine-X") is one of the most popular web servers in the world. It is used by millions of websites including Netflix, Airbnb, and WordPress.com. Nginx can serve several roles:

1. **Web Server** — Serve static files (HTML, CSS, JS, images) directly to users
2. **Reverse Proxy** — Forward incoming requests to another server (like Gunicorn) running behind it
3. **Load Balancer** — Distribute traffic across multiple application servers
4. **SSL Termination** — Handle HTTPS encryption/decryption

In this chapter, we use Nginx as a **reverse proxy** for our Flask application.

### What Is a Reverse Proxy?

A **reverse proxy** is a server that sits between the client (user's browser) and the application server (Gunicorn). The client never communicates directly with Gunicorn — all communication goes through Nginx.

```
Without Nginx (Chapter 7):

  Browser ──────────────────────────► Gunicorn:8000 ──► Flask App
  (User)           Direct connection


With Nginx (This Chapter):

  Browser ──► Nginx:80 ──► Gunicorn:8000 ──► Flask App
  (User)      (Reverse     (Application
               Proxy)       Server)
```

### Why Use Nginx in Front of Gunicorn?

| Benefit | Explanation |
|---------|-------------|
| **Standard port** | Nginx listens on port 80 (HTTP default), so users do not need to type a port number |
| **Static file serving** | Nginx serves CSS, JS, and images much faster than Gunicorn/Flask |
| **Security** | Nginx buffers requests, protecting Gunicorn from slow clients and certain types of attacks |
| **Connection handling** | Nginx efficiently handles thousands of simultaneous connections; Gunicorn can focus on running Python code |
| **SSL/HTTPS** | Nginx can handle HTTPS encryption, so Gunicorn does not need to deal with certificates |
| **Flexibility** | You can easily add caching, rate limiting, logging, and other features at the Nginx level |

### How the Full Architecture Works

```
┌─────────────────────────────────────────────────────────┐
│                    Ubuntu VM                         │
│                                                         │
│  ┌─────────────┐      ┌─────────────┐      ┌────────┐  │
│  │             │      │             │      │        │  │
│  │   Nginx     │─────►│  Gunicorn   │─────►│ Flask  │  │
│  │  (Port 80)  │      │ (Port 8000) │      │  App   │  │
│  │             │◄─────│             │◄─────│        │  │
│  └──────┬──────┘      └─────────────┘      └────────┘  │
│         │                                               │
│    Listens for                                          │
│    incoming traffic                                     │
│    from the internet                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
          ▲
          │  HTTP Request
          │
     ┌────┴────┐
     │ Browser │
     │ (User)  │
     └─────────┘
```

**Request flow:**
1. User types `http://192.168.1.50` in the browser
2. Browser sends request to port 80 (HTTP default)
3. **Nginx** receives the request on port 80
4. Nginx **forwards** the request to Gunicorn on `http://127.0.0.1:8000`
5. Gunicorn passes it to the **Flask application**
6. Flask processes the request and creates a response
7. Response travels back: Flask → Gunicorn → Nginx → Browser

### Nginx Configuration File Structure

Nginx uses configuration files to define how it handles traffic:

```
/etc/nginx/
├── nginx.conf              ← Main configuration file
├── sites-available/        ← All site configuration files
│   ├── default             ← Default site (comes with Nginx)
│   └── flaskapp            ← Our configuration (we will create this)
├── sites-enabled/          ← Active site configurations (symlinks)
│   └── flaskapp → ../sites-available/flaskapp
└── conf.d/                 ← Additional configuration snippets
```

**How it works:**
- You create a configuration file in `sites-available/`
- You create a symbolic link (shortcut) from `sites-enabled/` to activate it
- Nginx reads the files in `sites-enabled/` to know which sites to serve

---

## 8.3 Step-by-Step Instructions

Connect to your Ubuntu VM via SSH from your Windows terminal:

```cmd
ssh ubuntu@192.168.1.50
```

### Step 1: Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

### Step 2: Verify Nginx Is Running

```bash
sudo systemctl status nginx
```

You should see:
```
● nginx.service - A high performance web server and a reverse proxy server
     Active: active (running)
```

Press `q` to exit the status view.

### Step 3: Test Default Nginx Page

Open a browser on Windows and go to:

```
http://192.168.1.50
```

You should see the **"Welcome to nginx!"** default page. This confirms Nginx is installed and serving content on port 80.

> **If you cannot access it:** You need to allow HTTP traffic through the firewall:
> ```bash
> sudo ufw allow 'Nginx HTTP'
> sudo ufw status
> ```

### Step 4: Create the Flask App Configuration

Create a new Nginx configuration file for your Flask application:

```bash
sudo nano /etc/nginx/sites-available/flaskapp
```

Type the following configuration (copy it carefully):

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Save and exit nano:** Press `Ctrl + X`, then `Y`, then `Enter`.

**What each line does:**

| Line | Meaning |
|------|---------|
| `server { }` | Defines a server block — a set of rules for handling requests |
| `listen 80;` | Listen for incoming HTTP requests on port 80 |
| `server_name _;` | Match any hostname (the underscore `_` is a catch-all). In production, you would put your domain name here (e.g., `myapp.com`). |
| `location / { }` | Rules for all URLs starting with `/` (which means all URLs) |
| `proxy_pass http://127.0.0.1:8000;` | Forward all requests to Gunicorn running on port 8000 on localhost |
| `proxy_set_header Host $host;` | Pass the original hostname to Gunicorn (so Flask knows what domain was requested) |
| `proxy_set_header X-Real-IP $remote_addr;` | Pass the client's real IP address to Gunicorn (otherwise Flask would see 127.0.0.1 for every request) |
| `proxy_set_header X-Forwarded-For ...;` | Standard proxy header for tracking the chain of proxies |
| `proxy_set_header X-Forwarded-Proto $scheme;` | Tell Flask whether the original request was HTTP or HTTPS |

### Step 5: Remove the Default Configuration

The default Nginx configuration serves the "Welcome to nginx!" page. We need to disable it so our Flask app takes over:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

> **Note:** This removes the **symlink** in `sites-enabled`, not the actual file in `sites-available`. The default config file is still there if you ever need it.

### Step 6: Enable the Flask App Configuration

Create a symbolic link from `sites-available` to `sites-enabled`:

```bash
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
```

**What this does:**
- Creates a shortcut (`symlink`) in `sites-enabled/` that points to your config file in `sites-available/`
- Nginx reads `sites-enabled/` to know which sites are active
- To disable a site later, you just delete the symlink (not the original file)

### Step 7: Test the Configuration

Before restarting Nginx, test the configuration for syntax errors:

```bash
sudo nginx -t
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If you see errors, go back to Step 4 and check your configuration file for typos.

### Step 8: Restart Nginx

```bash
sudo systemctl restart nginx
```

Verify it is running:

```bash
sudo systemctl status nginx
```

### Step 9: Start Gunicorn

Nginx is now configured to forward traffic to port 8000, but Gunicorn needs to be running there:

```bash
cd ~/your-flask-app
source venv/bin/activate
gunicorn --bind 127.0.0.1:8000 --workers 3 app:app
```

> **Note:** This time we bind to `127.0.0.1:8000` (localhost only), not `0.0.0.0:8000`. Since Nginx handles all external traffic, Gunicorn only needs to accept connections from Nginx (which runs on the same machine).

### Step 10: Test the Full Setup

Open your browser on Windows and go to:

```
http://192.168.1.50
```

**Notice:** No `:8000` this time! Nginx is listening on port 80 (the default HTTP port) and forwarding requests to Gunicorn on port 8000 behind the scenes.

You should see your Flask application's response.

**Test from the command line (alternative):**

Open a new SSH session and run:

```bash
curl http://localhost
```

Or from Windows Command Prompt:

```cmd
curl http://192.168.1.50
```

---

## 8.4 Complete Setup Verification

Let's verify the entire chain is working:

```bash
# 1. Check Nginx is running
sudo systemctl status nginx

# 2. Check Nginx is listening on port 80
sudo ss -tuln | grep :80

# 3. Check Gunicorn is running on port 8000
sudo ss -tuln | grep :8000

# 4. Check the firewall allows HTTP traffic
sudo ufw status

# 5. Test the application
curl http://localhost
```

**Expected states:**

| Component | Status |
|-----------|--------|
| Nginx | Active, listening on port 80 |
| Gunicorn | Active, listening on port 8000 (localhost only) |
| UFW | Active, allowing ports 22 (SSH) and 80 (HTTP) |
| Application | Returns expected response to HTTP requests |

---

## 8.5 Self-Study Prompts

### Prompt 1: What is a reverse proxy and how does Nginx act as one?

<details>
<summary>Click to see a sample answer</summary>

A **reverse proxy** is a server that sits between clients (users) and backend servers (application servers). The client sends requests to the reverse proxy, which then forwards them to the appropriate backend server. The client does not know about or communicate directly with the backend server.

**Nginx acts as a reverse proxy by:**

1. Listening on port 80 for incoming HTTP requests from users
2. Receiving the request and reading the `proxy_pass` directive to determine where to forward it
3. Forwarding the request to Gunicorn at `http://127.0.0.1:8000`
4. Receiving Gunicorn's response
5. Sending the response back to the user

**Why "reverse"?** A regular (forward) proxy sits in front of clients and hides their identity from servers. A reverse proxy sits in front of servers and hides the server's identity from clients. The user thinks they are communicating with Nginx, but Nginx is secretly passing everything to Gunicorn.

**Benefits:**
- Users access port 80 (standard) instead of port 8000
- Gunicorn is not exposed to the internet directly
- Nginx efficiently handles connection management, static files, and buffering
- The architecture can be scaled by adding more Gunicorn instances behind Nginx

</details>

### Prompt 2: Why is Nginx used in production deployments?

<details>
<summary>Click to see a sample answer</summary>

Nginx is used in production for several reasons:

1. **Performance:** Nginx uses an asynchronous, event-driven architecture that handles thousands of simultaneous connections with minimal memory usage. Unlike traditional web servers that create a new process/thread per connection, Nginx uses a small number of worker processes that each handle many connections.

2. **Static file serving:** Nginx serves static files (CSS, JavaScript, images) extremely fast without involving the Python application. This reduces load on Gunicorn and Flask.

3. **Security:** Nginx buffers incoming requests, protecting backend servers from slow clients (slowloris attacks). It can also limit request rates, block IP addresses, and handle SSL/TLS encryption.

4. **Stability:** If Gunicorn crashes or needs to restart, Nginx can show a friendly error page instead of a connection error. It also handles graceful reloads without dropping active connections.

5. **SSL/HTTPS:** Nginx handles SSL certificate management and HTTPS encryption. Gunicorn communicates with Nginx over unencrypted HTTP locally, avoiding the overhead of encrypting internal traffic.

6. **Load Balancing:** In larger deployments, Nginx can distribute traffic across multiple Gunicorn instances running on different servers.

7. **Logging and Monitoring:** Nginx provides detailed access logs and error logs, useful for monitoring traffic patterns and debugging issues.

The Gunicorn documentation itself recommends placing Nginx in front:
> "We strongly advise you to use Nginx."

</details>

---

## 8.6 Activity

### Instructions

1. Install Nginx on your Ubuntu VM
2. Create the Nginx configuration file for your Flask app
3. Enable the configuration and restart Nginx
4. Start Gunicorn bound to localhost:8000
5. Access the application from your browser at `http://<your_vm_ip>` (no port number)
6. Take a screenshot showing either:
   - The application in your browser with the URL visible (showing no `:8000`), or
   - The terminal output of `curl http://localhost`

### Submission

- **Upload your screenshot**
- **Caption:** `Nginx configured successfully`

---

## 8.7 Troubleshooting

| Problem | Solution |
|---------|----------|
| Browser shows "Welcome to nginx!" instead of Flask app | You did not remove the default site. Run: `sudo rm /etc/nginx/sites-enabled/default && sudo systemctl restart nginx` |
| Browser shows "502 Bad Gateway" | Gunicorn is not running. Start it with: `cd ~/your-flask-app && source venv/bin/activate && gunicorn --bind 127.0.0.1:8000 app:app` |
| `nginx -t` shows configuration errors | Check `/etc/nginx/sites-available/flaskapp` for typos. Common mistakes: missing semicolons at the end of lines, mismatched curly braces. |
| Browser shows "403 Forbidden" | Nginx cannot access the application files. Check file permissions (Chapter 9 covers this). |
| Cannot reach `http://192.168.1.50` at all | Allow HTTP through the firewall: `sudo ufw allow 'Nginx HTTP'` |
| The symlink command fails | Make sure the file `/etc/nginx/sites-available/flaskapp` exists. Use `ls /etc/nginx/sites-available/` to verify. |
| Everything works via SSH but not from browser | Check if the VM is in Bridged mode. If using NAT, set up port forwarding: Host port 8080 → Guest port 80. |

---

## 8.8 Summary

In this chapter you:

- Installed Nginx on Ubuntu VM
- Learned what a reverse proxy is and why it is used
- Created an Nginx configuration file to proxy requests to Gunicorn
- Enabled the site configuration and tested it
- Verified the full deployment chain: Browser → Nginx (port 80) → Gunicorn (port 8000) → Flask App

Your Flask application is now accessible on the standard HTTP port without users needing to know about Gunicorn or port 8000.

**Next:** [Chapter 9 — User Management](chapter-09-user-management.md)
