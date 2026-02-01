# WSL2 + DevOps Beginner Guide 🚀
## From Zero to Deployment-Ready

> **For:** Complete beginners to DevOps  
> **Time:** 2-3 hours to complete  
> **Result:** A fully configured development environment ready for deployment learning

---

## 📋 Table of Contents

1. [What is WSL2 and Why?](#what-is-wsl2-and-why)
2. [Phase 1: Install WSL2](#phase-1-install-wsl2)
3. [Phase 2: First Steps in Linux](#phase-2-first-steps-in-linux)
4. [Phase 3: Essential Tools Installation](#phase-3-essential-tools-installation)
5. [Phase 4: Understanding the Terminal](#phase-4-understanding-the-terminal)
6. [Phase 5: Git & SSH Setup](#phase-5-git--ssh-setup)
7. [Phase 6: Docker in WSL2](#phase-6-docker-in-wsl2)
8. [Phase 7: Node.js Development Environment](#phase-7-nodejs-development-environment)
9. [Phase 8: Your First "Server"](#phase-8-your-first-server)
10. [Phase 9: Understanding Nginx](#phase-9-understanding-nginx)
11. [Phase 10: SSL/HTTPS Concepts](#phase-10-sslhttps-concepts)
12. [What's Next: Real Deployment](#whats-next-real-deployment)
13. [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## What is WSL2 and Why?

### The Problem
- Windows is great for development, but servers run Linux
- Learning Linux traditionally required a separate computer or VM
- Docker and deployment tools work better on Linux

### The Solution: WSL2
**Windows Subsystem for Linux 2** runs a real Linux kernel inside Windows.

```
┌─────────────────────────────────────────────────────────────┐
│                     YOUR WINDOWS PC                          │
│                                                              │
│   ┌─────────────────┐     ┌─────────────────────────────┐   │
│   │   Windows Apps  │     │        WSL2 (Ubuntu)        │   │
│   │                 │     │                             │   │
│   │   VS Code       │────▶│  Linux Terminal            │   │
│   │   Chrome        │     │  Node.js, Docker           │   │
│   │   Your IDE      │     │  Nginx, PostgreSQL         │   │
│   │                 │     │  Same as a real server!    │   │
│   └─────────────────┘     └─────────────────────────────┘   │
│                                                              │
│   Files at: \\wsl$\Ubuntu\home\username                     │
└─────────────────────────────────────────────────────────────┘
```

### Why Learn This?
1. **Same environment as servers** - What works in WSL2 works on AWS/DigitalOcean
2. **Learn Linux commands** - Essential for any DevOps role
3. **Run Docker natively** - Much faster than Docker Desktop on Windows
4. **Free practice environment** - Break things without consequences

---

## Phase 1: Install WSL2

### Step 1.1: Open PowerShell as Administrator

```
Press Windows key → Type "PowerShell" → Right-click → "Run as Administrator"
```

### Step 1.2: Install WSL2

```powershell
# This single command installs WSL2 with Ubuntu
wsl --install
```

**What this does:**
- Enables WSL feature
- Enables Virtual Machine Platform
- Downloads Ubuntu Linux
- Sets WSL2 as default

### Step 1.3: Restart Your Computer

```powershell
# You MUST restart for changes to take effect
shutdown /r /t 0
```

### Step 1.4: Complete Ubuntu Setup

After restart, Ubuntu will open automatically. If not:
- Press Windows key → Type "Ubuntu" → Open it

**You'll be asked to create:**
```
Enter new UNIX username: yourname
New password: ********
Retype new password: ********
```

> ⚠️ **IMPORTANT:** 
> - Username: lowercase, no spaces (e.g., `ali`, `devuser`)
> - Password: You won't see it as you type (that's normal!)
> - Remember this password - you'll need it for `sudo` commands

### Step 1.5: Verify Installation

```bash
# Check WSL version (should say "2")
wsl --version

# From inside Ubuntu, check you're running Ubuntu
cat /etc/os-release
```

**Expected output:**
```
NAME="Ubuntu"
VERSION="22.04.x LTS"
```

### Troubleshooting Installation

**If you get "WSL 2 requires an update to its kernel component":**
```powershell
# Download and install the WSL2 kernel update from:
# https://aka.ms/wsl2kernel
```

**If you get virtualization errors:**
1. Restart computer
2. Enter BIOS (usually F2, F12, or Del during boot)
3. Enable "Intel VT-x" or "AMD-V" (Virtualization)
4. Save and restart

---

## Phase 2: First Steps in Linux

### Step 2.1: Update Your System (ALWAYS Do This First!)

```bash
# Update package lists (like refreshing app store)
sudo apt update

# Upgrade installed packages (install updates)
sudo apt upgrade -y
```

**What's happening:**
- `sudo` = "Super User DO" - run command as administrator
- `apt` = Package manager (like Windows Store but for command line)
- `-y` = Auto-answer "yes" to prompts

### Step 2.2: Navigate Your New Linux World

```bash
# Where am I?
pwd
# Output: /home/yourname

# What's in this folder?
ls
# Output: (probably empty for now)

# List ALL files (including hidden)
ls -la
# Output: Shows files starting with . (dot files are hidden)

# Go to home directory (two ways)
cd ~
cd /home/yourname

# Go up one directory
cd ..

# Go to root of filesystem
cd /
```

### Step 2.3: Create Your First Files and Folders

```bash
# Go home
cd ~

# Create a projects folder
mkdir projects

# Go into it
cd projects

# Create a file
touch hello.txt

# Write something to it
echo "Hello from Linux!" > hello.txt

# Read it
cat hello.txt

# Create nested folders
mkdir -p learning/devops/day1

# See the tree structure
ls -R learning
```

### Step 2.4: Access Windows Files from Linux

```bash
# Your Windows C: drive is mounted at /mnt/c
cd /mnt/c/Users/YourWindowsUsername/Desktop

# List files on your Windows desktop
ls

# Go back to Linux home
cd ~
```

### Step 2.5: Access Linux Files from Windows

Open Windows Explorer and type in address bar:
```
\\wsl$\Ubuntu\home\yourname
```

You can now drag/drop files between Windows and Linux!

---

## Phase 3: Essential Tools Installation

### Step 3.1: Install Build Essentials

```bash
# Essential compilation tools
sudo apt install -y build-essential

# Common utilities
sudo apt install -y curl wget git vim nano unzip zip
```

**What each tool does:**
| Tool | Purpose |
|------|---------|
| `curl` | Download files from URLs |
| `wget` | Download files (simpler than curl) |
| `git` | Version control (you'll use this constantly) |
| `vim` | Terminal text editor (advanced) |
| `nano` | Terminal text editor (beginner-friendly) |
| `unzip` | Extract .zip files |

### Step 3.2: Install Node.js (via NVM - Recommended)

```bash
# Install NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Reload your terminal configuration
source ~/.bashrc

# Verify NVM installed
nvm --version

# Install Node.js LTS (Long Term Support)
nvm install --lts

# Verify Node installed
node --version
npm --version
```

**Why NVM?**
- Easily switch between Node versions
- No permission issues with global packages
- Industry standard approach

### Step 3.3: Install pnpm (Faster npm)

```bash
# Install pnpm globally
npm install -g pnpm

# Verify
pnpm --version
```

### Step 3.4: Install Useful CLI Tools

```bash
# Tree - visualize directory structure
sudo apt install -y tree

# htop - better process viewer
sudo apt install -y htop

# jq - JSON processor
sudo apt install -y jq

# net-tools - networking utilities
sudo apt install -y net-tools
```

### Step 3.5: Verify Everything

```bash
# Run this verification script
echo "=== Verification ==="
echo "Node: $(node --version)"
echo "npm: $(npm --version)"
echo "pnpm: $(pnpm --version)"
echo "Git: $(git --version)"
echo "=== All Good! ==="
```

---

## Phase 4: Understanding the Terminal

### Essential Commands You'll Use Daily

```bash
# ┌─────────────────────────────────────────────────────────┐
# │              NAVIGATION                                  │
# └─────────────────────────────────────────────────────────┘

pwd                    # Print Working Directory (where am I?)
ls                     # List files
ls -la                 # List ALL files with details
cd folder              # Change directory
cd ..                  # Go up one level
cd ~                   # Go to home directory
cd -                   # Go to previous directory

# ┌─────────────────────────────────────────────────────────┐
# │              FILE OPERATIONS                             │
# └─────────────────────────────────────────────────────────┘

touch file.txt         # Create empty file
mkdir folder           # Create folder
mkdir -p a/b/c         # Create nested folders
cp file.txt copy.txt   # Copy file
cp -r folder/ copy/    # Copy folder recursively
mv old.txt new.txt     # Move/rename file
rm file.txt            # Delete file (NO UNDO!)
rm -rf folder/         # Delete folder (DANGEROUS - NO UNDO!)

# ┌─────────────────────────────────────────────────────────┐
# │              VIEWING FILES                               │
# └─────────────────────────────────────────────────────────┘

cat file.txt           # Print entire file
head file.txt          # First 10 lines
tail file.txt          # Last 10 lines
tail -f file.txt       # Follow file (watch for changes)
less file.txt          # Scrollable view (q to quit)

# ┌─────────────────────────────────────────────────────────┐
# │              SEARCHING                                   │
# └─────────────────────────────────────────────────────────┘

grep "text" file.txt   # Find text in file
grep -r "text" folder/ # Find text in all files in folder
find . -name "*.js"    # Find files by name

# ┌─────────────────────────────────────────────────────────┐
# │              PERMISSIONS                                 │
# └─────────────────────────────────────────────────────────┘

chmod +x script.sh     # Make file executable
chmod 755 script.sh    # rwxr-xr-x (owner can all, others read/execute)
chmod 644 file.txt     # rw-r--r-- (owner read/write, others read)
chown user:group file  # Change ownership

# ┌─────────────────────────────────────────────────────────┐
# │              PROCESS MANAGEMENT                          │
# └─────────────────────────────────────────────────────────┘

ps aux                 # List all processes
htop                   # Interactive process viewer
kill PID               # Kill process by ID
killall node           # Kill all processes named "node"
```

### Keyboard Shortcuts That Save Time

```
Ctrl + C         Stop current command
Ctrl + D         Exit terminal / EOF
Ctrl + L         Clear screen (same as 'clear')
Ctrl + R         Search command history
Ctrl + A         Go to beginning of line
Ctrl + E         Go to end of line
Tab              Auto-complete file/folder names
Tab Tab          Show all auto-complete options
↑ / ↓            Navigate command history
```

### The Pipe (`|`) - Chain Commands Together

```bash
# Count number of files
ls | wc -l

# Find process using port 3000
ps aux | grep node

# Sort files by size
ls -la | sort -k5 -n

# Chain multiple commands
cat file.txt | grep "error" | wc -l
# Read file → find lines with "error" → count them
```

### Redirection (`>`, `>>`, `<`)

```bash
# Write output to file (overwrites!)
echo "Hello" > file.txt

# Append output to file
echo "World" >> file.txt

# Read input from file
sort < unsorted.txt

# Redirect errors to file
npm install 2> errors.log

# Redirect both output and errors
npm install > output.log 2>&1
```

---

## Phase 5: Git & SSH Setup

### Step 5.1: Configure Git

```bash
# Set your identity (use your GitHub email)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Enable colors
git config --global color.ui auto

# Verify
git config --list
```

### Step 5.2: Generate SSH Key

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "your.email@example.com"

# Press Enter to accept default location (~/.ssh/id_ed25519)
# Enter passphrase (optional but recommended)
```

**What gets created:**
```
~/.ssh/
├── id_ed25519       # Private key (NEVER share this!)
└── id_ed25519.pub   # Public key (safe to share)
```

### Step 5.3: Add SSH Key to GitHub

```bash
# Copy your public key
cat ~/.ssh/id_ed25519.pub
```

1. Go to [github.com/settings/keys](https://github.com/settings/keys)
2. Click "New SSH Key"
3. Title: "WSL2 Ubuntu"
4. Key: Paste the output from above
5. Click "Add SSH Key"

### Step 5.4: Test GitHub Connection

```bash
# Test SSH connection
ssh -T git@github.com

# You should see:
# Hi username! You've successfully authenticated...
```

### Step 5.5: Clone a Repository

```bash
cd ~/projects

# Clone via SSH (recommended)
git clone git@github.com:username/repo.git

# Or clone via HTTPS
git clone https://github.com/username/repo.git
```

### Essential Git Commands

```bash
# ┌─────────────────────────────────────────────────────────┐
# │              DAILY GIT WORKFLOW                          │
# └─────────────────────────────────────────────────────────┘

git status                    # What's changed?
git add .                     # Stage all changes
git add file.txt              # Stage specific file
git commit -m "message"       # Commit with message
git push                      # Push to remote
git pull                      # Pull from remote

# ┌─────────────────────────────────────────────────────────┐
# │              BRANCHES                                    │
# └─────────────────────────────────────────────────────────┘

git branch                    # List branches
git branch feature            # Create branch
git checkout feature          # Switch to branch
git checkout -b feature       # Create AND switch
git merge feature             # Merge branch into current

# ┌─────────────────────────────────────────────────────────┐
# │              HISTORY & UNDO                              │
# └─────────────────────────────────────────────────────────┘

git log --oneline             # Compact history
git diff                      # See unstaged changes
git stash                     # Save changes temporarily
git stash pop                 # Restore stashed changes
```

---

## Phase 6: Docker in WSL2

### Why Docker?
Docker lets you run applications in isolated "containers" - like mini virtual machines but faster.

```
Traditional:                    With Docker:
┌─────────────────────┐        ┌─────────────────────┐
│  Your Computer      │        │  Your Computer      │
│  ┌───────────────┐  │        │  ┌───────────────┐  │
│  │ App 1 (Node 14)│  │        │  │ Container 1   │  │
│  │ App 2 (Node 16)│  │  ──▶   │  │ Node 14 + App1│  │
│  │ (CONFLICT!)    │  │        │  ├───────────────┤  │
│  └───────────────┘  │        │  │ Container 2   │  │
│                     │        │  │ Node 16 + App2│  │
└─────────────────────┘        │  └───────────────┘  │
                               └─────────────────────┘
                               (No conflicts!)
```

### Step 6.1: Install Docker in WSL2

```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc 2>/dev/null

# Install prerequisites
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add your user to docker group (no sudo needed for docker commands)
sudo usermod -aG docker $USER

# IMPORTANT: Log out and back in, or run:
newgrp docker
```

### Step 6.2: Verify Docker Works

```bash
# Check Docker version
docker --version

# Run hello-world container
docker run hello-world

# You should see:
# Hello from Docker!
# This message shows that your installation appears to be working correctly.
```

### Step 6.3: Essential Docker Commands

```bash
# ┌─────────────────────────────────────────────────────────┐
# │              IMAGES                                      │
# └─────────────────────────────────────────────────────────┘

docker images                 # List downloaded images
docker pull nginx             # Download image
docker rmi nginx              # Remove image

# ┌─────────────────────────────────────────────────────────┐
# │              CONTAINERS                                  │
# └─────────────────────────────────────────────────────────┘

docker ps                     # List running containers
docker ps -a                  # List ALL containers
docker run nginx              # Run container
docker run -d nginx           # Run in background (detached)
docker run -p 8080:80 nginx   # Map port 8080 to container's 80
docker stop container_id      # Stop container
docker rm container_id        # Remove container

# ┌─────────────────────────────────────────────────────────┐
# │              DOCKER COMPOSE                              │
# └─────────────────────────────────────────────────────────┘

docker compose up             # Start all services
docker compose up -d          # Start in background
docker compose down           # Stop all services
docker compose logs           # View logs
docker compose ps             # List services
```

### Step 6.4: Your First Docker Container

```bash
# Run Nginx web server
docker run -d -p 8080:80 --name my-nginx nginx

# Open in browser: http://localhost:8080
# You should see "Welcome to nginx!"

# Check running containers
docker ps

# View logs
docker logs my-nginx

# Stop and remove
docker stop my-nginx
docker rm my-nginx
```

---

## Phase 7: Node.js Development Environment

### Step 7.1: Create a Sample Project

```bash
cd ~/projects

# Create project folder
mkdir my-first-server
cd my-first-server

# Initialize Node project
npm init -y

# Install Express
npm install express
```

### Step 7.2: Create a Simple Server

```bash
# Create the server file
cat > server.js << 'EOF'
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ 
    message: 'Hello from my first server!',
    timestamp: new Date().toISOString()
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
EOF
```

### Step 7.3: Run Your Server

```bash
# Start the server
node server.js

# Open new terminal (or Ctrl+C to stop, test, then restart)
# Visit: http://localhost:3000
```

### Step 7.4: Add Start Script to package.json

```bash
# Edit package.json
nano package.json
```

Add to "scripts":
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  }
}
```

Now you can run:
```bash
npm start    # Production mode
npm run dev  # Development mode with auto-restart
```

---

## Phase 8: Your First "Server"

### Understanding Client-Server Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     THE INTERNET                             │
│                                                              │
│   ┌──────────┐        HTTP Request        ┌──────────────┐  │
│   │  CLIENT  │  ──────────────────────▶  │    SERVER    │  │
│   │ (Browser)│                            │   (Node.js)  │  │
│   │          │  ◀──────────────────────  │              │  │
│   └──────────┘        HTTP Response       └──────────────┘  │
│                                                              │
│   Request: GET /api/users                                   │
│   Response: { "users": [...] }                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### What Happens When You Visit a Website

```
1. You type: https://example.com/api/users
                    │
                    ▼
2. DNS Lookup: example.com → 93.184.216.34
                    │
                    ▼
3. TCP Connection to 93.184.216.34:443 (HTTPS = port 443)
                    │
                    ▼
4. TLS Handshake (encryption established)
                    │
                    ▼
5. HTTP Request sent:
   GET /api/users HTTP/1.1
   Host: example.com
                    │
                    ▼
6. Server processes request, queries database
                    │
                    ▼
7. HTTP Response:
   HTTP/1.1 200 OK
   Content-Type: application/json
   {"users": [...]}
                    │
                    ▼
8. Browser displays data
```

### Important Ports to Know

| Port | Protocol | Use |
|------|----------|-----|
| 22 | SSH | Remote terminal access |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 3000 | - | Common Node.js dev port |
| 5432 | PostgreSQL | Database |
| 6379 | Redis | Cache |
| 5672 | RabbitMQ | Message queue |

---

## Phase 9: Understanding Nginx

### What is Nginx?

Nginx (pronounced "engine-x") is a **reverse proxy** and **web server**.

```
Without Nginx (Development):
┌──────────┐       ┌──────────────┐
│  Browser │ ────▶ │ Node.js:3000 │
└──────────┘       └──────────────┘

With Nginx (Production):
┌──────────┐       ┌───────────┐       ┌──────────────┐
│  Browser │ ────▶ │  Nginx:80 │ ────▶ │ Node.js:3000 │
└──────────┘       │   :443    │       └──────────────┘
                   └───────────┘
                        │
                   ┌────┴────┐
                   │ Benefits│
                   ├─────────┤
                   │ SSL/TLS │
                   │ Caching │
                   │ Gzip    │
                   │ Load    │
                   │ Balance │
                   └─────────┘
```

### Step 9.1: Install Nginx

```bash
# Install Nginx
sudo apt install -y nginx

# Start Nginx
sudo systemctl start nginx

# Enable on boot
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

### Step 9.2: Visit Default Page

Open browser: http://localhost

You should see: "Welcome to nginx!"

### Step 9.3: Understand Nginx Files

```bash
# Main configuration
cat /etc/nginx/nginx.conf

# Site configurations
ls /etc/nginx/sites-available/
ls /etc/nginx/sites-enabled/

# Default site
cat /etc/nginx/sites-available/default
```

### Step 9.4: Create a Reverse Proxy

```bash
# Create new site configuration
sudo nano /etc/nginx/sites-available/myapp
```

Paste this:
```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

Now:
- Start your Node.js server on port 3000
- Visit http://localhost (port 80)
- Nginx forwards to your Node.js app!

---

## Phase 10: SSL/HTTPS Concepts

### Why HTTPS?

```
HTTP (Insecure):
┌────────┐         ┌────────┐         ┌────────┐
│ Client │ ──────▶ │ Hacker │ ──────▶ │ Server │
└────────┘         └────────┘         └────────┘
            "password123"      "password123"
            (readable!)        

HTTPS (Secure):
┌────────┐         ┌────────┐         ┌────────┐
│ Client │ ──────▶ │ Hacker │ ──────▶ │ Server │
└────────┘         └────────┘         └────────┘
            "x#$@!&*%..."     "x#$@!&*%..."
            (encrypted!)      (decrypted at server)
```

### How SSL/TLS Works (Simplified)

```
1. CLIENT HELLO
   Client: "I want to connect securely. I support TLS 1.3"
                              │
                              ▼
2. SERVER HELLO + CERTIFICATE
   Server: "Let's use TLS 1.3. Here's my certificate proving I'm example.com"
                              │
                              ▼
3. CLIENT VERIFIES CERTIFICATE
   Client: "Certificate is signed by trusted authority. I trust you."
                              │
                              ▼
4. KEY EXCHANGE
   Both parties derive a shared secret key (without sending it!)
                              │
                              ▼
5. ENCRYPTED COMMUNICATION
   All data is now encrypted with the shared key
```

### Certificate Types

| Type | Validation | Cost | Use Case |
|------|-----------|------|----------|
| **DV** (Domain Validation) | Proves domain ownership | Free (Let's Encrypt) | Most websites |
| **OV** (Organization Validation) | Verifies business | $50-200/yr | Business sites |
| **EV** (Extended Validation) | Extensive verification | $200-500/yr | Banks, enterprise |

### Let's Encrypt (Free SSL!)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get certificate (on real server with domain)
sudo certbot --nginx -d yourdomain.com

# Auto-renewal is set up automatically!
# Test it with:
sudo certbot renew --dry-run
```

### Local Development (Self-Signed)

For local testing only:
```bash
# Generate self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
```

---

## What's Next: Real Deployment

You've learned the fundamentals! Here's your learning path:

### Immediate Next Steps

```
1. Practice Phase
   └── Build 2-3 small projects locally using these skills

2. Cloud Account Setup
   └── Create free accounts:
       • Vercel (frontend)
       • Render (backend)
       • Neon (database)

3. First Deployment
   └── Follow: docs/deployment/QUICK_DEPLOY.md
       Deploy your CareCircle project!

4. Real VPS Experience
   └── Follow: docs/extras/learning/engineering-mastery/DEVOPS/Complete-vps-setup-guide.md
       Set up a real server from scratch
```

### Learning Resources in This Project

| Document | What You'll Learn |
|----------|-------------------|
| `QUICK_DEPLOY.md` | Deploy to production in 90 minutes |
| `FREE_DEPLOYMENT_GUIDE.md` | Detailed free-tier deployment |
| `Complete-vps-setup-guide.md` | Full VPS setup from scratch |
| `devop-complete-guide.md` | Comprehensive DevOps reference |
| `ssl-tls-complete-guide.md` | Deep SSL/TLS knowledge |

---

## Quick Reference Cheatsheet

### Commands to Remember

```bash
# ═══════════════════════════════════════════════════════════
#                    SYSTEM
# ═══════════════════════════════════════════════════════════
sudo apt update && sudo apt upgrade -y    # Update system
sudo systemctl status nginx               # Check service status
sudo systemctl restart nginx              # Restart service

# ═══════════════════════════════════════════════════════════
#                    NAVIGATION
# ═══════════════════════════════════════════════════════════
pwd                 # Where am I?
ls -la              # List all files
cd ~/projects       # Go to projects
cd -                # Go to previous directory

# ═══════════════════════════════════════════════════════════
#                    FILES
# ═══════════════════════════════════════════════════════════
cat file.txt        # Read file
nano file.txt       # Edit file (Ctrl+O save, Ctrl+X exit)
tail -f log.txt     # Follow file (watch for changes)

# ═══════════════════════════════════════════════════════════
#                    GIT
# ═══════════════════════════════════════════════════════════
git status          # What's changed?
git add .           # Stage all
git commit -m "msg" # Commit
git push            # Push to GitHub
git pull            # Pull from GitHub

# ═══════════════════════════════════════════════════════════
#                    DOCKER
# ═══════════════════════════════════════════════════════════
docker ps           # List running containers
docker compose up -d   # Start services
docker compose down    # Stop services
docker compose logs    # View logs

# ═══════════════════════════════════════════════════════════
#                    NODE.JS
# ═══════════════════════════════════════════════════════════
nvm install --lts   # Install Node LTS
npm start           # Start app
npm run dev         # Start in dev mode

# ═══════════════════════════════════════════════════════════
#                    NGINX
# ═══════════════════════════════════════════════════════════
sudo nginx -t                    # Test config
sudo systemctl reload nginx      # Apply changes
sudo certbot --nginx -d domain   # Get SSL certificate
```

### Important Paths

```
~/.ssh/                    # SSH keys
~/projects/                # Your projects
/etc/nginx/                # Nginx configuration
/var/log/nginx/            # Nginx logs
/var/www/                  # Web server files (traditional)
```

---

## Congratulations! 🎉

You now have:
- ✅ WSL2 with Ubuntu running
- ✅ Essential tools installed (Git, Node.js, Docker)
- ✅ Understanding of terminal commands
- ✅ SSH and Git configured
- ✅ Docker working
- ✅ Basic understanding of Nginx and SSL
- ✅ A path forward for real deployment

**Your next step:** Open `docs/deployment/QUICK_DEPLOY.md` and deploy your project!

---

*Need help? Check `docs/runbooks/COMMON_ISSUES.md` for troubleshooting.*

