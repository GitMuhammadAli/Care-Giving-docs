# Complete VPS Setup Guide 🚀
## From SSH Key Generation to Production-Ready Server

> **Author:** DevOps Reference Guide  
> **Last Updated:** January 2026  
> **Server OS:** Ubuntu 24.04 LTS  
> **Domain Example:** jabwewed.com

---

## 📋 Table of Contents

1. [Credentials Table](#-credentials-table---fill-first)
2. [Phase 1: SSH Key Generation](#phase-1-ssh-key-generation)
3. [Phase 2: Create VPS (DigitalOcean/AWS/Other)](#phase-2-create-vps)
4. [Phase 3: First SSH Connection](#phase-3-first-ssh-connection)
5. [Phase 4: System Update & Timezone](#phase-4-system-update--timezone)
6. [Phase 5: Firewall Setup (UFW)](#phase-5-firewall-setup-ufw)
7. [Phase 6: SSH Security Hardening](#phase-6-ssh-security-hardening)
8. [Phase 7: Create Non-Root User](#phase-7-create-non-root-user)
9. [Phase 8: Install Tech Stack](#phase-8-install-tech-stack)
10. [Phase 9: Configure Nginx](#phase-9-configure-nginx)
11. [Phase 10: SSL Certificate (Let's Encrypt)](#phase-10-ssl-certificate-lets-encrypt)
12. [Phase 11: Fail2Ban (Brute Force Protection)](#phase-11-fail2ban-brute-force-protection)
13. [Phase 12: Nginx Security Headers](#phase-12-nginx-security-headers)
14. [Phase 13: Deploy Node.js App with PM2](#phase-13-deploy-nodejs-app-with-pm2)
15. [Phase 14: Nginx Reverse Proxy](#phase-14-nginx-reverse-proxy)
16. [Quick Reference Commands](#-quick-reference-commands)
17. [Troubleshooting Guide](#-troubleshooting-guide)
18. [Security Checklist](#-final-security-checklist)

---

## 📝 Credentials Table - Fill First!

| Field | Your Value | Example |
|-------|------------|---------|
| Server IP | `_______________` | 147.182.195.236 |
| Domain Name | `_______________` | jabwewed.com |
| Username | `_______________` | usama |
| Email | `_______________` | admin@jabwewed.com |
| App Port | `_______________` | 3000 |
| Project Name | `_______________` | ems-be |
| Project Path | `_______________` | /var/www/ems-be |
| Git Repo URL | `_______________` | github.com/user/repo |

---

## Phase 1: SSH Key Generation

> **📍 Location:** Your LOCAL machine (Windows/Mac/Linux)

SSH keys provide secure, passwordless authentication. Generate these BEFORE creating your VPS.

### Option A: Windows (PowerShell)

```powershell
# Check if SSH is available
ssh -V

# Generate SSH key pair (Ed25519 - most secure, recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# OR use RSA if Ed25519 not supported (older systems)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**Prompts:**
- **File location:** Press Enter (use default `~/.ssh/id_ed25519`)
- **Passphrase:** Enter a password (optional but recommended)
- **Confirm:** Re-enter passphrase

### Option B: Mac/Linux (Terminal)

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "your_email@example.com"

# Set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### View Your Public Key

```bash
# Windows (PowerShell)
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub

# Mac/Linux
cat ~/.ssh/id_ed25519.pub
```

**Output looks like:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your_email@example.com
```

> ⚠️ **Copy this public key** - You'll need it when creating your VPS!

### SSH Key Locations

| Key Type | Location | Share? |
|----------|----------|--------|
| Private Key | `~/.ssh/id_ed25519` | ❌ NEVER share! |
| Public Key | `~/.ssh/id_ed25519.pub` | ✅ Safe to share |

### 💡 Alternative: Generate Key with Custom Name

```bash
# Custom name for multiple servers
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/myserver_key
```

---

## Phase 2: Create VPS

### Option A: DigitalOcean (Dashboard Method)

1. Login to [DigitalOcean](https://cloud.digitalocean.com)
2. Click **Create → Droplets**
3. Choose:
   - **Region:** Closest to your users (e.g., Singapore, Frankfurt)
   - **Image:** Ubuntu 24.04 LTS
   - **Size:** Basic $6/month (1GB RAM) for small projects
   - **Authentication:** Select "SSH Keys"
4. Click **New SSH Key**
5. Paste your public key (from Phase 1)
6. Give it a name (e.g., "My Laptop")
7. Click **Create Droplet**
8. **Copy the IP address** shown after creation

### Option B: DigitalOcean (CLI Method - doctl)

```bash
# Install doctl (DigitalOcean CLI)
# Mac
brew install doctl

# Windows (Chocolatey)
choco install doctl

# Linux
snap install doctl

# Authenticate
doctl auth init
# Enter your API token from DigitalOcean dashboard

# Create droplet with SSH key
doctl compute droplet create my-server \
  --region sgp1 \
  --size s-1vcpu-1gb \
  --image ubuntu-24-04-x64 \
  --ssh-keys $(doctl compute ssh-key list --format ID --no-header)

# Get your server IP
doctl compute droplet list --format Name,PublicIPv4
```

### Option C: AWS EC2

```bash
# Using AWS CLI
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name your-key-pair-name \
  --security-group-ids sg-xxxxxxxx

# Get IP
aws ec2 describe-instances --query 'Reservations[*].Instances[*].PublicIpAddress'
```

### Option D: Linode/Vultr/Other

Similar process - create instance, select Ubuntu 24.04, add SSH key, note the IP.

### 🔍 Get Server IP via Terminal (After Creation)

```bash
# DigitalOcean
doctl compute droplet list

# AWS
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,PublicIpAddress]' --output table

# Generic - if you have hostname
nslookup your-server-hostname
dig your-server-hostname +short
```

---

## Phase 3: First SSH Connection

### Connect to Server

```bash
# Basic connection
ssh root@YOUR_SERVER_IP

# Example
ssh root@147.182.195.236

# If using custom key location
ssh -i ~/.ssh/custom_key root@YOUR_SERVER_IP

# If using non-standard port
ssh -p 2222 root@YOUR_SERVER_IP
```

**First time connection:**
- You'll see "authenticity can't be established" warning
- Type `yes` and press Enter
- This adds server to your known_hosts file

**Success looks like:**
```
root@ubuntu:~#
```

### Verify Server Details

```bash
# Check Ubuntu version
lsb_release -a

# Check server resources
free -h          # RAM
df -h            # Disk space
nproc            # CPU cores

# Check current date/timezone
date
timedatectl

# Check server IP (from server itself)
curl -4 ifconfig.me
# OR
hostname -I
```

### 💡 Alternative: SSH Config File (Easier Connection)

Create `~/.ssh/config` on your LOCAL machine:

```
Host myserver
    HostName 147.182.195.236
    User root
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```

Now connect with just:
```bash
ssh myserver
```

---

## Phase 4: System Update & Timezone

### Update System Packages

```bash
# Update package list
apt update

# Upgrade installed packages
apt upgrade -y

# Remove unnecessary packages
apt autoremove -y

# If system shows "restart required"
reboot
```

> ⚠️ After reboot, wait 30-60 seconds then SSH back in.

### Set Timezone

```bash
# List available timezones
timedatectl list-timezones | grep Asia

# Set timezone (Pakistan example)
timedatectl set-timezone Asia/Karachi

# Verify
date
timedatectl
```

**Common Timezones:**
| Region | Timezone |
|--------|----------|
| Pakistan | Asia/Karachi |
| India | Asia/Kolkata |
| UAE | Asia/Dubai |
| UK | Europe/London |
| US East | America/New_York |
| US West | America/Los_Angeles |

### Enable Automatic Security Updates

```bash
apt install unattended-upgrades -y
dpkg-reconfigure -plow unattended-upgrades
```

---

## Phase 5: Firewall Setup (UFW)

> **UFW = Uncomplicated Firewall** - Controls which ports are accessible

### ⚠️ CRITICAL: Allow SSH First!

```bash
# Check UFW status
ufw status

# Allow SSH FIRST (or you'll be locked out!)
ufw allow 22/tcp

# Allow HTTP and HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Rate limit SSH (blocks IPs with too many attempts)
ufw limit 22/tcp

# Enable firewall
ufw enable

# Verify rules
ufw status verbose
```

**Expected Output:**
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22/tcp                     LIMIT IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
```

### UFW Commands Reference

| Action | Command |
|--------|---------|
| Allow specific port | `ufw allow 8080/tcp` |
| Allow port range | `ufw allow 3000:3010/tcp` |
| Allow from specific IP | `ufw allow from 192.168.1.100` |
| Allow from IP to port | `ufw allow from 192.168.1.100 to any port 22` |
| Delete a rule | `ufw delete allow 8080/tcp` |
| Delete by number | `ufw status numbered` then `ufw delete 3` |
| Disable firewall | `ufw disable` |
| Reset all rules | `ufw reset` |
| Reload rules | `ufw reload` |

### 💡 Alternative: iptables (Advanced)

```bash
# View current rules
iptables -L -n -v

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### 💡 Alternative: nftables (Modern)

```bash
# nftables is successor to iptables (Ubuntu 24.04 has it)
apt install nftables -y
systemctl enable nftables

# Basic config at /etc/nftables.conf
```

> **Recommendation:** Stick with UFW unless you need advanced features.

---

## Phase 6: SSH Security Hardening

### Edit SSH Configuration

```bash
# Backup original
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Edit config
nano /etc/ssh/sshd_config
```

**Find and modify these lines:**

```bash
# Disable password authentication (key-only)
PasswordAuthentication no

# Disable empty passwords
PermitEmptyPasswords no

# Allow only key-based root login (or 'no' after creating another user)
PermitRootLogin prohibit-password

# Enable public key authentication
PubkeyAuthentication yes

# Limit authentication attempts
MaxAuthTries 3

# Disconnect idle sessions (5 minutes)
ClientAliveInterval 300
ClientAliveCountMax 0

# Disable X11 forwarding (not needed for servers)
X11Forwarding no

# (Optional) Change default port
# Port 2222
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Apply Changes

```bash
# Test configuration (important!)
sshd -t

# Restart SSH service
systemctl restart ssh

# Verify SSH is running
systemctl status ssh
```

> ⚠️ **Keep your current SSH session open!** Open a NEW terminal and test login before closing.

### 💡 Alternative: Change SSH Port

```bash
# In sshd_config
Port 2222

# Update firewall BEFORE restarting SSH
ufw allow 2222/tcp
ufw delete allow 22/tcp

# Restart SSH
systemctl restart ssh

# Connect with new port
ssh -p 2222 root@YOUR_SERVER_IP
```

---

## Phase 7: Create Non-Root User

> **Why?** Running as root is dangerous. One wrong command can destroy everything.

### Create User

```bash
# Create user (replace 'usama' with your username)
adduser usama

# You'll be prompted for:
# - Password
# - Full name (optional, press Enter)
# - Other info (optional, press Enter)

# Add to sudo group
usermod -aG sudo usama

# Verify user created
id usama
```

### Setup SSH for New User

```bash
# Create SSH directory
mkdir -p /home/usama/.ssh

# Copy authorized keys from root
cp /root/.ssh/authorized_keys /home/usama/.ssh/

# Set correct ownership
chown -R usama:usama /home/usama/.ssh

# Set correct permissions
chmod 700 /home/usama/.ssh
chmod 600 /home/usama/.ssh/authorized_keys
```

### Test New User Login

```bash
# From your LOCAL machine (new terminal)
ssh usama@YOUR_SERVER_IP

# Test sudo access
sudo whoami
# Enter your password
# Output should be: root
```

### (Optional) Disable Root Login Completely

After confirming new user works:

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Change this line
PermitRootLogin no

# Restart SSH
sudo systemctl restart ssh
```

### 💡 Alternative: Add Existing Public Key for User

```bash
# If you have a different key for this user
echo "ssh-ed25519 AAAAC3... user@email.com" >> /home/usama/.ssh/authorized_keys
```

---

## Phase 8: Install Tech Stack

### Node.js & NPM

```bash
# Method 1: NodeSource (Recommended - Latest LTS)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt install -y nodejs

# Verify
node -v    # Should show v20.x.x
npm -v     # Should show 10.x.x
```

**Alternative Methods:**

```bash
# Method 2: NVM (Node Version Manager) - Multiple versions
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20

# Method 3: Ubuntu default (may be older version)
apt install nodejs npm -y
```

### Nginx (Web Server)

```bash
# Install
apt install nginx -y

# Enable on boot
systemctl enable nginx

# Start
systemctl start nginx

# Verify
systemctl status nginx
nginx -v

# Test in browser: http://YOUR_SERVER_IP
```

### Redis (Cache/Session Store)

```bash
# Install
apt install redis-server -y

# Enable on boot
systemctl enable redis-server

# Start
systemctl start redis-server

# Test
redis-cli ping
# Output: PONG
```

### Other Essential Tools

```bash
# Git
apt install git -y
git --version

# Build essentials (for native npm modules)
apt install build-essential -y

# Certbot (SSL)
apt install certbot python3-certbot-nginx -y

# Process manager
npm install -g pm2

# (Optional) htop - better process viewer
apt install htop -y
```

### 💡 Alternative: Docker Installation

```bash
# Install Docker
curl -fsSL https://get.docker.com | sh

# Add user to docker group
usermod -aG docker usama

# Install Docker Compose
apt install docker-compose-plugin -y

# Verify
docker --version
docker compose version
```

---

## Phase 9: Configure Nginx

### Create Project Directory

```bash
# Create directory
mkdir -p /var/www/ems-be

# Set ownership (use your username)
chown -R usama:usama /var/www/ems-be

# Navigate
cd /var/www/ems-be
```

### Create Nginx Configuration

```bash
# Create config file
nano /etc/nginx/sites-available/jabwewed.com
```

**Paste this configuration:**

```nginx
##
# Nginx configuration for jabwewed.com
##

# Redirect HTTP to HTTPS (will work after SSL setup)
server {
    listen 80;
    listen [::]:80;
    server_name jabwewed.com www.jabwewed.com;

    # For Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    # Redirect to HTTPS (uncomment after SSL is set up)
    # return 301 https://$host$request_uri;

    # Temporary: Proxy to app (remove after SSL)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Enable Site

```bash
# Create symbolic link
ln -s /etc/nginx/sites-available/jabwewed.com /etc/nginx/sites-enabled/

# Remove default site (optional)
rm -f /etc/nginx/sites-enabled/default

# Test configuration
nginx -t

# Reload Nginx
systemctl reload nginx
```

---

## Phase 10: SSL Certificate (Let's Encrypt)

### Prerequisites Check

```bash
# Verify DNS points to your server
dig jabwewed.com +short
# Should return your server IP

# Check HTTP works
curl -I http://jabwewed.com
# Should return 200 OK or 502 (if app not running)
```

### Get SSL Certificate

```bash
# Run Certbot with Nginx plugin
certbot --nginx -d jabwewed.com -d www.jabwewed.com
```

**Prompts:**
1. **Email:** Enter your email (for renewal notifications)
2. **Terms of Service:** Type `Y`
3. **Share email with EFF:** Type `N` (optional)
4. **Redirect HTTP to HTTPS:** Choose `2` (recommended)

**Success Output:**
```
Congratulations! You have successfully enabled HTTPS on https://jabwewed.com
Certificate is saved at: /etc/letsencrypt/live/jabwewed.com/fullchain.pem
This certificate expires on 2026-04-09.
```

### Verify SSL

```bash
# Test HTTPS
curl -I https://jabwewed.com

# Check certificate details
certbot certificates
```

### Test Auto-Renewal

```bash
# Dry run test
certbot renew --dry-run

# Check renewal timer
systemctl status certbot.timer

# View timer schedule
systemctl list-timers | grep certbot
```

### (Optional) Custom Renewal Script

```bash
# Create script
nano /usr/local/bin/ssl-renew.sh
```

```bash
#!/bin/bash
LOG="/var/log/ssl-renewal.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "[$DATE] Starting SSL renewal check..." >> $LOG
certbot renew --quiet --post-hook "systemctl reload nginx"

if [ $? -eq 0 ]; then
    echo "[$DATE] Renewal successful" >> $LOG
else
    echo "[$DATE] Renewal FAILED!" >> $LOG
fi
```

```bash
# Make executable
chmod +x /usr/local/bin/ssl-renew.sh

# Add to crontab
crontab -e

# Add this line (runs daily at 3 AM)
0 3 * * * /usr/local/bin/ssl-renew.sh
```

### 💡 Alternative: Cloudflare SSL

1. Add domain to Cloudflare
2. Update nameservers at registrar
3. Enable "Full (Strict)" SSL in Cloudflare dashboard
4. Cloudflare handles SSL automatically

---

## Phase 11: Fail2Ban (Brute Force Protection)

### Install Fail2Ban

```bash
apt install fail2ban -y
```

### Configure Fail2Ban

```bash
# Create local config (don't edit jail.conf directly)
nano /etc/fail2ban/jail.local
```

**Paste:**

```ini
[DEFAULT]
# Ban for 1 hour
bantime = 1h

# Check last 10 minutes
findtime = 10m

# Ban after 5 failed attempts
maxretry = 5

# Use UFW for banning
banaction = ufw

# Email notifications (optional)
# destemail = your@email.com
# sendername = Fail2Ban
# action = %(action_mwl)s

[sshd]
enabled = true
port = 22
maxretry = 3
bantime = 24h

[nginx-http-auth]
enabled = true

[nginx-limit-req]
enabled = true

[nginx-botsearch]
enabled = true
```

### Start Fail2Ban

```bash
# Restart to apply config
systemctl restart fail2ban

# Enable on boot
systemctl enable fail2ban

# Check status
systemctl status fail2ban

# Check active jails
fail2ban-client status

# Check specific jail
fail2ban-client status sshd
```

### Fail2Ban Commands

| Action | Command |
|--------|---------|
| View all jails | `fail2ban-client status` |
| View banned IPs (SSH) | `fail2ban-client status sshd` |
| Unban an IP | `fail2ban-client set sshd unbanip 192.168.1.100` |
| Ban an IP manually | `fail2ban-client set sshd banip 192.168.1.100` |
| Check fail2ban log | `tail -f /var/log/fail2ban.log` |

---

## Phase 12: Nginx Security Headers

### Update Nginx Config

```bash
nano /etc/nginx/sites-available/jabwewed.com
```

**Replace entire file with:**

```nginx
##
# Nginx configuration for jabwewed.com
##

# Redirect HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name jabwewed.com www.jabwewed.com;
    return 301 https://$host$request_uri;
}

# Main HTTPS server
server {
    listen 443 ssl;
    listen [::]:443 ssl ipv6only=on;
    server_name jabwewed.com www.jabwewed.com;

    # SSL Certificates (managed by Certbot)
    ssl_certificate /etc/letsencrypt/live/jabwewed.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jabwewed.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

    # Hide Nginx version
    server_tokens off;

    # Root directory (for static files)
    root /var/www/html;
    index index.html index.htm;

    # Proxy to Node.js app
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }

    # (Optional) Static files location
    # location /static/ {
    #     alias /var/www/ems-be/public/;
    #     expires 30d;
    #     add_header Cache-Control "public, immutable";
    # }
}
```

### Apply Changes

```bash
# Test config
nginx -t

# Reload
systemctl reload nginx

# Verify headers
curl -I https://jabwewed.com
```

### Security Headers Explained

| Header | Purpose |
|--------|---------|
| `X-Frame-Options` | Prevents clickjacking attacks |
| `X-Content-Type-Options` | Prevents MIME type sniffing |
| `X-XSS-Protection` | Enables browser XSS filter |
| `Referrer-Policy` | Controls referrer information |
| `Strict-Transport-Security` | Forces HTTPS for future requests |
| `Permissions-Policy` | Controls browser features |
| `server_tokens off` | Hides Nginx version |

---

## Phase 13: Deploy Node.js App with PM2

### Setup Application

```bash
# Switch to your user
su - usama

# Go to project directory
cd /var/www/ems-be

# Clone your repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git .

# OR if repo already exists
git pull origin main

# Install dependencies
npm install

# Create environment file
nano .env
```

**.env Example:**
```env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://localhost:27017/mydb
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-super-secret-key-change-this
```

### Start with PM2

```bash
# Start application
pm2 start app.js --name "ems-be"

# OR with npm script
pm2 start npm --name "ems-be" -- start

# OR with ecosystem file (recommended)
pm2 start ecosystem.config.js
```

### PM2 Ecosystem File (Recommended)

Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'ems-be',
    script: 'app.js',
    instances: 'max',        // Use all CPU cores
    exec_mode: 'cluster',    // Cluster mode
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    // Restart if memory exceeds 500MB
    max_memory_restart: '500M',
    // Logs
    log_file: '/var/log/pm2/ems-be.log',
    error_file: '/var/log/pm2/ems-be-error.log',
    out_file: '/var/log/pm2/ems-be-out.log',
    // Restart on file changes (dev only)
    watch: false,
    // Restart delay
    restart_delay: 4000
  }]
};
```

```bash
# Start with ecosystem
pm2 start ecosystem.config.js --env production
```

### PM2 Startup (Auto-restart on Reboot)

```bash
# Generate startup script
pm2 startup

# Copy and run the command PM2 outputs
# Example: sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u usama --hp /home/usama

# Save current processes
pm2 save
```

### PM2 Commands Reference

| Action | Command |
|--------|---------|
| Start app | `pm2 start app.js --name "myapp"` |
| Stop app | `pm2 stop myapp` |
| Restart app | `pm2 restart myapp` |
| Reload (0 downtime) | `pm2 reload myapp` |
| Delete app | `pm2 delete myapp` |
| View all apps | `pm2 list` |
| View logs | `pm2 logs myapp` |
| View logs (all) | `pm2 logs` |
| Monitor | `pm2 monit` |
| Show details | `pm2 show myapp` |
| Restart on file change | `pm2 start app.js --watch` |

### 💡 Alternative: Using systemd Service

Create `/etc/systemd/system/ems-be.service`:

```ini
[Unit]
Description=EMS Backend Node.js App
After=network.target

[Service]
Type=simple
User=usama
WorkingDirectory=/var/www/ems-be
ExecStart=/usr/bin/node app.js
Restart=on-failure
RestartSec=10
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable ems-be
systemctl start ems-be
systemctl status ems-be
```

---

## Phase 14: Nginx Reverse Proxy

### What is Reverse Proxy?

```
Internet → Nginx (Port 80/443) → Node.js App (Port 3000)
```

- Nginx handles SSL termination
- Nginx serves static files efficiently
- Nginx load balances multiple app instances
- Node.js only handles application logic

### Basic Reverse Proxy (Already in Phase 12)

The config in Phase 12 already includes reverse proxy. Key parts:

```nginx
location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### Advanced: Multiple Apps / Subdomains

```nginx
# API subdomain
server {
    listen 443 ssl;
    server_name api.jabwewed.com;

    ssl_certificate /etc/letsencrypt/live/jabwewed.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jabwewed.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        # ... proxy headers
    }
}

# Admin subdomain
server {
    listen 443 ssl;
    server_name admin.jabwewed.com;

    ssl_certificate /etc/letsencrypt/live/jabwewed.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jabwewed.com/privkey.pem;

    location / {
        proxy_pass http://localhost:4000;
        # ... proxy headers
    }
}
```

### Load Balancing Multiple Instances

```nginx
upstream nodejs_cluster {
    least_conn;  # Send to least connected server
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 443 ssl;
    server_name jabwewed.com;

    location / {
        proxy_pass http://nodejs_cluster;
        # ... proxy headers
    }
}
```

### WebSocket Support

```nginx
location /socket.io/ {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}
```

---

## 📋 Quick Reference Commands

### Service Management

| Action | Command |
|--------|---------|
| Start service | `systemctl start SERVICE` |
| Stop service | `systemctl stop SERVICE` |
| Restart service | `systemctl restart SERVICE` |
| Reload config | `systemctl reload SERVICE` |
| Check status | `systemctl status SERVICE` |
| Enable on boot | `systemctl enable SERVICE` |
| Disable on boot | `systemctl disable SERVICE` |
| View logs | `journalctl -u SERVICE -f` |

### Nginx Commands

| Action | Command |
|--------|---------|
| Test config | `nginx -t` |
| Reload | `systemctl reload nginx` |
| View error logs | `tail -f /var/log/nginx/error.log` |
| View access logs | `tail -f /var/log/nginx/access.log` |

### SSL Commands

| Action | Command |
|--------|---------|
| View certificates | `certbot certificates` |
| Test renewal | `certbot renew --dry-run` |
| Force renewal | `certbot renew --force-renewal` |
| Add new domain | `certbot --nginx -d newdomain.com` |

### Firewall Commands

| Action | Command |
|--------|---------|
| Check status | `ufw status verbose` |
| Allow port | `ufw allow 8080/tcp` |
| Delete rule | `ufw delete allow 8080/tcp` |
| View numbered | `ufw status numbered` |

### PM2 Commands

| Action | Command |
|--------|---------|
| List all | `pm2 list` |
| Logs | `pm2 logs` |
| Monitor | `pm2 monit` |
| Restart all | `pm2 restart all` |
| Save state | `pm2 save` |

### System Diagnostics

| Action | Command |
|--------|---------|
| Disk space | `df -h` |
| Memory | `free -h` |
| CPU/Processes | `htop` |
| Open ports | `ss -tulpn` |
| Network connections | `netstat -an` |
| Running processes | `ps aux` |

---

## 🔧 Troubleshooting Guide

### Cannot SSH into Server

```bash
# Check if SSH is running (from VPS console)
systemctl status ssh

# Check firewall
ufw status

# Allow SSH
ufw allow 22/tcp

# Check SSH config for errors
sshd -t

# View SSH logs
tail -f /var/log/auth.log
```

### 502 Bad Gateway

```bash
# Check if app is running
pm2 status

# Check app logs
pm2 logs

# Check if port is correct in Nginx
grep proxy_pass /etc/nginx/sites-available/jabwewed.com

# Check if app is listening
ss -tlpn | grep 3000

# Restart everything
pm2 restart all
systemctl restart nginx
```

### SSL Certificate Issues

```bash
# Check certificate
certbot certificates

# Check DNS
dig jabwewed.com +short

# Test renewal
certbot renew --dry-run

# Check Nginx SSL config
nginx -t

# View Certbot logs
tail -f /var/log/letsencrypt/letsencrypt.log
```

### Permission Denied

```bash
# Fix ownership
chown -R usama:usama /var/www/ems-be

# Fix permissions
chmod -R 755 /var/www/ems-be

# Check SELinux (if enabled)
getenforce
```

### High Memory Usage

```bash
# Check memory
free -h

# Find memory-hungry processes
ps aux --sort=-%mem | head -20

# PM2 memory restart
pm2 restart myapp --max-memory-restart 500M

# Clear system cache
sync; echo 3 > /proc/sys/vm/drop_caches
```

### Nginx Config Errors

```bash
# Test config
nginx -t

# Common issues:
# - Missing semicolon at end of line
# - Missing closing brace }
# - Duplicate server_name
# - Invalid SSL certificate path

# Check syntax highlighting in nano
nano /etc/nginx/sites-available/jabwewed.com
```

---

## ✅ Final Security Checklist

### Server Security

- [ ] SSH key authentication only (password disabled)
- [ ] Non-root user created with sudo access
- [ ] Root login disabled or restricted
- [ ] SSH on non-standard port (optional)
- [ ] UFW firewall enabled (ports 22, 80, 443 only)
- [ ] Fail2Ban installed and configured
- [ ] Automatic security updates enabled

### Web Security

- [ ] SSL/TLS certificate installed
- [ ] HTTP redirects to HTTPS
- [ ] SSL auto-renewal working
- [ ] Security headers configured
- [ ] Nginx version hidden
- [ ] Hidden files blocked

### Application Security

- [ ] Environment variables in .env (not in code)
- [ ] .env file not in git repository
- [ ] Dependencies up to date (`npm audit`)
- [ ] PM2 running in cluster mode
- [ ] Application logs configured
- [ ] Error handling implemented

### Monitoring (Optional)

- [ ] Server monitoring (DigitalOcean/AWS built-in)
- [ ] Application monitoring (PM2 Plus / NewRelic)
- [ ] Uptime monitoring (UptimeRobot / Pingdom)
- [ ] Log aggregation (Papertrail / Loggly)

---

## 🎉 Congratulations!

Your server is now production-ready with:

```
┌─────────────────────────────────────────────┐
│         YOUR_DOMAIN.com - SECURED! 🛡️        │
├─────────────────────────────────────────────┤
│  ✅ SSL Certificate     (Auto-renewing)     │
│  ✅ Firewall            (UFW active)        │
│  ✅ Brute Force Block   (Fail2Ban active)   │
│  ✅ Security Headers    (All enabled)       │
│  ✅ Non-Root User       (Sudo access)       │
│  ✅ Process Manager     (PM2 running)       │
│  ✅ Reverse Proxy       (Nginx configured)  │
└─────────────────────────────────────────────┘
```

---

## 📚 Additional Resources

- [DigitalOcean Tutorials](https://www.digitalocean.com/community/tutorials)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [SSL Labs Test](https://www.ssllabs.com/ssltest/) - Test your SSL configuration

---

> **Created with ❤️ for Usama**  
> **Last Updated:** January 2026

---

# PART 2: DEVOPS DEEP DIVE

---

## Docker & Containerization

### What is Docker?

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCKER CONCEPTS                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Image = Blueprint (like a class)                           │
│  Container = Running instance (like an object)              │
│                                                             │
│  ┌─────────────┐     ┌─────────────┐                       │
│  │   IMAGE     │     │  CONTAINER  │                       │
│  │  (nginx)    │ ──► │  (running)  │                       │
│  │             │     │   nginx     │                       │
│  │  Read-only  │     │  Has state  │                       │
│  └─────────────┘     └─────────────┘                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Install Docker

```bash
# Quick install (recommended)
curl -fsSL https://get.docker.com | sh

# Add user to docker group (no sudo needed)
sudo usermod -aG docker $USER

# Logout and login, then verify
docker --version
docker run hello-world
```

### Docker Commands Cheatsheet

```bash
# === IMAGES ===
docker images                     # List images
docker pull nginx:latest          # Download image
docker build -t myapp:v1 .        # Build from Dockerfile
docker tag myapp:v1 myapp:latest  # Tag image
docker push username/myapp:v1     # Push to registry
docker rmi nginx                  # Remove image
docker image prune -a             # Remove unused images

# === CONTAINERS ===
docker ps                         # Running containers
docker ps -a                      # All containers
docker run -d --name web nginx    # Run detached
docker stop web                   # Stop container
docker start web                  # Start container
docker restart web                # Restart container
docker rm web                     # Remove container
docker rm -f web                  # Force remove

# === LOGS & EXEC ===
docker logs web                   # View logs
docker logs -f web                # Follow logs
docker logs --tail 100 web        # Last 100 lines
docker exec -it web bash          # Enter container
docker exec web ls /etc           # Run command

# === INSPECT ===
docker inspect web                # Full details
docker stats                      # Resource usage
docker top web                    # Running processes

# === CLEANUP ===
docker system prune               # Remove unused data
docker system prune -a --volumes  # Remove everything unused
```

### Dockerfile for Node.js

```dockerfile
# === BUILD STAGE ===
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files first (better caching)
COPY package*.json ./

# Install ALL dependencies (including dev)
RUN npm ci

# Copy source code
COPY . .

# Build if needed (TypeScript, etc.)
RUN npm run build

# === PRODUCTION STAGE ===
FROM node:20-alpine AS production

WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy package files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production && npm cache clean --force

# Copy built files from builder
COPY --from=builder /app/dist ./dist

# Change ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => r.statusCode === 200 ? process.exit(0) : process.exit(1))"

# Start with dumb-init
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/app.js"]
```

### Docker Compose

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  # Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: ems-be
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - backend

  # MongoDB
  mongo:
    image: mongo:7
    container_name: mongodb
    restart: unless-stopped
    volumes:
      - mongo-data:/data/db
      - ./mongo-init:/docker-entrypoint-initdb.d:ro
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  # Redis
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  # Nginx (Reverse Proxy)
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - app
    networks:
      - backend
      - frontend

volumes:
  mongo-data:
  redis-data:

networks:
  backend:
    driver: bridge
  frontend:
    driver: bridge
```

### Docker Compose Commands

```bash
# Start services
docker-compose up -d

# Start with build
docker-compose up -d --build

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs -f
docker-compose logs -f app

# Scale service
docker-compose up -d --scale app=3

# Execute command
docker-compose exec app sh

# View status
docker-compose ps
```

### 📚 Docker Resources

| Resource | Link |
|----------|------|
| Docker Docs | https://docs.docker.com/ |
| Docker Hub | https://hub.docker.com/ |
| Play with Docker | https://labs.play-with-docker.com/ |
| Docker Curriculum | https://docker-curriculum.com/ |
| Dockerfile Best Practices | https://docs.docker.com/develop/develop-images/dockerfile_best-practices/ |

---

## CI/CD Pipelines

### What is CI/CD?

```
┌─────────────────────────────────────────────────────────────┐
│                    CI/CD PIPELINE                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CI = Continuous Integration                                │
│  • Automatically build code when pushed                     │
│  • Run tests automatically                                  │
│  • Catch bugs early                                         │
│                                                             │
│  CD = Continuous Deployment/Delivery                        │
│  • Automatically deploy to staging/production               │
│  • No manual deployment steps                               │
│                                                             │
│  ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐     │
│  │  Code  │───►│ Build  │───►│  Test  │───►│ Deploy │     │
│  │  Push  │    │        │    │        │    │        │     │
│  └────────┘    └────────┘    └────────┘    └────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### GitHub Actions

**.github/workflows/ci-cd.yml:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ===== TEST JOB =====
  test:
    name: Test
    runs-on: ubuntu-latest
    
    services:
      mongodb:
        image: mongo:7
        ports:
          - 27017:27017
      redis:
        image: redis:7
        ports:
          - 6379:6379
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
        env:
          DATABASE_URL: mongodb://localhost:27017/test
          REDIS_URL: redis://localhost:6379
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # ===== BUILD JOB =====
  build:
    name: Build Docker Image
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    
    permissions:
      contents: read
      packages: write
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix=
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ===== DEPLOY JOB =====
  deploy:
    name: Deploy to Production
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/ems-be
            
            # Pull latest code
            git pull origin main
            
            # Pull latest Docker image
            docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main
            
            # Restart services
            docker-compose pull
            docker-compose up -d
            
            # Cleanup old images
            docker image prune -f
            
            # Health check
            sleep 10
            curl -f http://localhost:3000/health || exit 1
      
      - name: Notify on success
        if: success()
        uses: slackapi/slack-github-action@v1.24.0
        with:
          payload: |
            {
              "text": "✅ Deployment successful! Version: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Notify on failure
        if: failure()
        uses: slackapi/slack-github-action@v1.24.0
        with:
          payload: |
            {
              "text": "❌ Deployment failed! Check: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Required GitHub Secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|--------|-------------|
| `SERVER_HOST` | Your server IP (147.182.195.236) |
| `SERVER_USER` | SSH username (usama) |
| `SSH_PRIVATE_KEY` | Your private SSH key |
| `SLACK_WEBHOOK` | Slack webhook URL (optional) |
| `CODECOV_TOKEN` | Codecov token (optional) |

### 📚 CI/CD Resources

| Resource | Link |
|----------|------|
| GitHub Actions Docs | https://docs.github.com/en/actions |
| GitHub Actions Marketplace | https://github.com/marketplace?type=actions |
| GitLab CI/CD | https://docs.gitlab.com/ee/ci/ |
| Jenkins | https://www.jenkins.io/doc/ |
| CircleCI | https://circleci.com/docs/ |

---

## Infrastructure as Code (IaC)

### Terraform Basics

**Install Terraform:**

```bash
# Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify
terraform --version
```

**main.tf - DigitalOcean Example:**

```hcl
# Provider
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

# Variables
variable "do_token" {
  description = "DigitalOcean API Token"
  sensitive   = true
}

variable "ssh_key_fingerprint" {
  description = "SSH Key Fingerprint"
}

variable "region" {
  default = "sgp1"
}

# Provider config
provider "digitalocean" {
  token = var.do_token
}

# Create VPC
resource "digitalocean_vpc" "main" {
  name     = "main-vpc"
  region   = var.region
  ip_range = "10.10.10.0/24"
}

# Create Droplet
resource "digitalocean_droplet" "web" {
  name     = "web-server"
  size     = "s-1vcpu-1gb"
  image    = "ubuntu-24-04-x64"
  region   = var.region
  vpc_uuid = digitalocean_vpc.main.id
  ssh_keys = [var.ssh_key_fingerprint]
  
  tags = ["web", "production"]
  
  # Run on creation
  user_data = <<-EOF
    #!/bin/bash
    apt update
    apt upgrade -y
    apt install -y nginx
    systemctl enable nginx
    systemctl start nginx
  EOF
}

# Create Firewall
resource "digitalocean_firewall" "web" {
  name        = "web-firewall"
  droplet_ids = [digitalocean_droplet.web.id]

  inbound_rule {
    protocol         = "tcp"
    port_range       = "22"
    source_addresses = ["0.0.0.0/0"]
  }

  inbound_rule {
    protocol         = "tcp"
    port_range       = "80"
    source_addresses = ["0.0.0.0/0"]
  }

  inbound_rule {
    protocol         = "tcp"
    port_range       = "443"
    source_addresses = ["0.0.0.0/0"]
  }

  outbound_rule {
    protocol              = "tcp"
    port_range            = "all"
    destination_addresses = ["0.0.0.0/0"]
  }

  outbound_rule {
    protocol              = "udp"
    port_range            = "all"
    destination_addresses = ["0.0.0.0/0"]
  }
}

# Output
output "droplet_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```

**Terraform Commands:**

```bash
# Initialize
terraform init

# Plan (preview changes)
terraform plan

# Apply (create resources)
terraform apply

# Destroy (delete resources)
terraform destroy

# Format code
terraform fmt

# Validate
terraform validate
```

### Ansible Basics

**Install Ansible:**

```bash
sudo apt install ansible -y
ansible --version
```

**inventory.ini:**

```ini
[webservers]
web1 ansible_host=147.182.195.236 ansible_user=root

[databases]
db1 ansible_host=147.182.195.237 ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

**playbook.yml:**

```yaml
---
- name: Setup Web Server
  hosts: webservers
  become: yes
  
  vars:
    node_version: "20"
    app_path: /var/www/ems-be
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install required packages
      apt:
        name:
          - nginx
          - git
          - curl
          - build-essential
        state: present
    
    - name: Install Node.js
      shell: |
        curl -fsSL https://deb.nodesource.com/setup_{{ node_version }}.x | bash -
        apt install -y nodejs
      args:
        creates: /usr/bin/node
    
    - name: Install PM2
      npm:
        name: pm2
        global: yes
    
    - name: Create app directory
      file:
        path: "{{ app_path }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Clone repository
      git:
        repo: https://github.com/user/repo.git
        dest: "{{ app_path }}"
        version: main
    
    - name: Install npm dependencies
      npm:
        path: "{{ app_path }}"
        production: yes
    
    - name: Start application with PM2
      shell: |
        cd {{ app_path }}
        pm2 start app.js --name ems-be
        pm2 save
        pm2 startup
      become_user: www-data
    
    - name: Configure Nginx
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Restart Nginx
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Run Ansible:**

```bash
# Test connection
ansible all -i inventory.ini -m ping

# Run playbook
ansible-playbook -i inventory.ini playbook.yml

# Dry run
ansible-playbook -i inventory.ini playbook.yml --check

# Verbose output
ansible-playbook -i inventory.ini playbook.yml -vvv
```

### 📚 IaC Resources

| Resource | Link |
|----------|------|
| Terraform Docs | https://www.terraform.io/docs |
| Terraform Registry | https://registry.terraform.io/ |
| Ansible Docs | https://docs.ansible.com/ |
| Ansible Galaxy | https://galaxy.ansible.com/ |
| Pulumi Docs | https://www.pulumi.com/docs/ |

---

# PART 3: CLOUD & ORCHESTRATION

---

## Kubernetes (K8s)

### What is Kubernetes?

```
┌─────────────────────────────────────────────────────────────┐
│                    KUBERNETES                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Container Orchestration = Managing many containers         │
│                                                             │
│  ┌─────────────── CLUSTER ───────────────────┐             │
│  │                                           │             │
│  │  ┌─── Node 1 ───┐   ┌─── Node 2 ───┐     │             │
│  │  │  ┌───────┐   │   │  ┌───────┐   │     │             │
│  │  │  │ Pod A │   │   │  │ Pod A │   │     │  Auto       │
│  │  │  └───────┘   │   │  └───────┘   │     │  Scaling    │
│  │  │  ┌───────┐   │   │  ┌───────┐   │     │             │
│  │  │  │ Pod B │   │   │  │ Pod C │   │     │  Self       │
│  │  │  └───────┘   │   │  └───────┘   │     │  Healing    │
│  │  └──────────────┘   └──────────────┘     │             │
│  │                                           │             │
│  └───────────────────────────────────────────┘             │
│                                                             │
│  K8s automatically:                                         │
│  • Scales pods up/down                                      │
│  • Restarts failed containers                               │
│  • Distributes load                                         │
│  • Manages secrets                                          │
│  • Handles networking                                       │
└─────────────────────────────────────────────────────────────┘
```

### Install kubectl

```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client
```

### Basic K8s Resources

**deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ems-be
  labels:
    app: ems-be
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ems-be
  template:
    metadata:
      labels:
        app: ems-be
    spec:
      containers:
        - name: ems-be
          image: ghcr.io/user/ems-be:latest
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
```

**service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ems-be-service
spec:
  selector:
    app: ems-be
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

**ingress.yaml:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ems-be-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - jabwewed.com
      secretName: jabwewed-tls
  rules:
    - host: jabwewed.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: ems-be-service
                port:
                  number: 80
```

### kubectl Commands

```bash
# === CLUSTER ===
kubectl cluster-info
kubectl get nodes

# === PODS ===
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- bash

# === DEPLOYMENTS ===
kubectl get deployments
kubectl apply -f deployment.yaml
kubectl scale deployment ems-be --replicas=5
kubectl rollout status deployment/ems-be
kubectl rollout history deployment/ems-be
kubectl rollout undo deployment/ems-be

# === SERVICES ===
kubectl get services
kubectl apply -f service.yaml
kubectl expose deployment ems-be --port=80 --target-port=3000

# === SECRETS ===
kubectl create secret generic app-secrets --from-literal=database-url=mongodb://...
kubectl get secrets
kubectl describe secret app-secrets
```

### 📚 Kubernetes Resources

| Resource | Link |
|----------|------|
| K8s Docs | https://kubernetes.io/docs/ |
| K8s Tutorials | https://kubernetes.io/docs/tutorials/ |
| Play with K8s | https://labs.play-with-k8s.com/ |
| Helm Charts | https://artifacthub.io/ |
| K8s Academy | https://www.youtube.com/kubernetesacademy |

---

## Monitoring & Observability

### Prometheus + Grafana Stack

```
┌─────────────────────────────────────────────────────────────┐
│                MONITORING STACK                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    Scrapes     ┌─────────────┐            │
│  │   Your App  │ ◄──────────────│ Prometheus  │            │
│  │   /metrics  │    metrics     │  (Storage)  │            │
│  └─────────────┘                └──────┬──────┘            │
│                                        │                    │
│  ┌─────────────┐                       │                    │
│  │   Node      │ ◄─────────────────────┤                    │
│  │   Exporter  │                       │                    │
│  └─────────────┘                       │                    │
│                                        ▼                    │
│                                 ┌─────────────┐            │
│                                 │   Grafana   │            │
│                                 │ (Dashboard) │            │
│                                 └─────────────┘            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Docker Compose for Monitoring

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - "9090:9090"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    ports:
      - "3001:3000"
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
```

**prometheus.yml:**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'app'
    static_configs:
      - targets: ['app:3000']
```

### 📚 Monitoring Resources

| Resource | Link |
|----------|------|
| Prometheus Docs | https://prometheus.io/docs/ |
| Grafana Docs | https://grafana.com/docs/ |
| Grafana Dashboards | https://grafana.com/grafana/dashboards/ |
| ELK Stack | https://www.elastic.co/what-is/elk-stack |

---

# PART 4: SYSTEM ARCHITECTURE

---

## Architecture Patterns

### Monolithic vs Microservices

```
┌─────────────────────────────────────────────────────────────┐
│            MONOLITHIC vs MICROSERVICES                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  MONOLITHIC:                 MICROSERVICES:                 │
│  ┌─────────────────┐         ┌─────┐ ┌─────┐ ┌─────┐       │
│  │     ONE BIG     │         │User │ │Order│ │Pay  │       │
│  │   APPLICATION   │         │Svc  │ │Svc  │ │Svc  │       │
│  │                 │         └──┬──┘ └──┬──┘ └──┬──┘       │
│  │  ┌───────────┐  │            │       │       │          │
│  │  │   Users   │  │            └───────┼───────┘          │
│  │  ├───────────┤  │                    │                  │
│  │  │  Orders   │  │            ┌───────▼───────┐          │
│  │  ├───────────┤  │            │  API Gateway  │          │
│  │  │ Payments  │  │            └───────────────┘          │
│  │  └───────────┘  │                                       │
│  └─────────────────┘                                       │
│                                                             │
│  ✅ Simple to start         ✅ Scale independently         │
│  ✅ Easy debugging          ✅ Tech flexibility            │
│  ❌ Hard to scale           ❌ Complex infrastructure      │
│  ❌ Single point of failure ❌ Network latency             │
└─────────────────────────────────────────────────────────────┘
```

### Scalability Patterns

```
┌─────────────────────────────────────────────────────────────┐
│              SCALABILITY PATTERNS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  VERTICAL SCALING:           HORIZONTAL SCALING:            │
│  (Scale Up)                  (Scale Out)                    │
│                                                             │
│  ┌─────────────┐             ┌─────┐ ┌─────┐ ┌─────┐       │
│  │             │             │App 1│ │App 2│ │App 3│       │
│  │  BIGGER     │             └──┬──┘ └──┬──┘ └──┬──┘       │
│  │  SERVER     │                └───────┼───────┘          │
│  │             │                        │                  │
│  │  More RAM   │                ┌───────▼───────┐          │
│  │  More CPU   │                │ Load Balancer │          │
│  │             │                └───────────────┘          │
│  └─────────────┘                                           │
│                                                             │
│  ❌ Has limits              ✅ Unlimited scaling           │
│  ❌ Single point of failure ✅ High availability           │
│  ✅ Simple                  ❌ More complex                │
└─────────────────────────────────────────────────────────────┘
```

### High Availability Architecture

```
┌─────────────────────────────────────────────────────────────┐
│            HIGH AVAILABILITY SETUP                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    ┌────────────┐                          │
│                    │    CDN     │                          │
│                    │(CloudFlare)│                          │
│                    └─────┬──────┘                          │
│                          │                                  │
│                    ┌─────▼──────┐                          │
│                    │   Load     │                          │
│                    │  Balancer  │                          │
│                    └─────┬──────┘                          │
│            ┌─────────────┼─────────────┐                   │
│            │             │             │                   │
│       ┌────▼────┐  ┌────▼────┐  ┌────▼────┐              │
│       │ App 1   │  │ App 2   │  │ App 3   │              │
│       │(Zone A) │  │(Zone B) │  │(Zone C) │              │
│       └────┬────┘  └────┬────┘  └────┬────┘              │
│            └─────────────┼─────────────┘                   │
│                          │                                  │
│              ┌───────────┼───────────┐                     │
│              │           │           │                     │
│         ┌────▼────┐ ┌────▼────┐ ┌────▼────┐               │
│         │Primary  │ │Replica  │ │Replica  │               │
│         │   DB    │ │   DB    │ │   DB    │               │
│         └─────────┘ └─────────┘ └─────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 📚 Architecture Resources

| Resource | Link |
|----------|------|
| System Design Primer | https://github.com/donnemartin/system-design-primer |
| Awesome Scalability | https://github.com/binhnguyennus/awesome-scalability |
| High Scalability Blog | http://highscalability.com/ |
| Martin Fowler | https://martinfowler.com/architecture/ |
| ByteByteGo | https://bytebytego.com/ |

---

# PART 5: DEVOPS CAREER ROADMAP

---

## Learning Path

```
┌─────────────────────────────────────────────────────────────┐
│                DEVOPS LEARNING ROADMAP                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  BEGINNER (0-6 months)                                      │
│  ├── Linux basics                    ✓ You are here!       │
│  ├── Networking fundamentals                                │
│  ├── Git & GitHub                                           │
│  ├── Bash scripting                                         │
│  └── Basic server setup (this guide)                        │
│                                                             │
│  INTERMEDIATE (6-12 months)                                 │
│  ├── Docker & Docker Compose                                │
│  ├── CI/CD (GitHub Actions/GitLab CI)                      │
│  ├── Cloud basics (AWS/GCP/Azure)                          │
│  ├── Infrastructure as Code (Terraform)                    │
│  └── Configuration management (Ansible)                    │
│                                                             │
│  ADVANCED (1-2 years)                                       │
│  ├── Kubernetes                                             │
│  ├── Service mesh (Istio)                                   │
│  ├── Monitoring (Prometheus/Grafana)                       │
│  ├── Logging (ELK Stack)                                    │
│  └── Security & compliance                                  │
│                                                             │
│  EXPERT (2+ years)                                          │
│  ├── Multi-cloud architecture                               │
│  ├── Platform engineering                                   │
│  ├── SRE practices                                          │
│  ├── Cost optimization                                      │
│  └── Team leadership                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Salary Expectations

| Level | Role | Pakistan (PKR/month) | US (USD/year) |
|-------|------|---------------------|---------------|
| Entry | Junior DevOps | 80K - 150K | $50K - $70K |
| Mid | DevOps Engineer | 200K - 400K | $90K - $130K |
| Senior | Sr. DevOps Engineer | 400K - 700K | $130K - $180K |
| Lead | DevOps Architect | 700K - 1.2M | $160K - $220K |
| Expert | Principal/SRE | 1M - 2M+ | $200K - $350K+ |

## 🎓 Certifications

| Certification | Provider | Difficulty |
|---------------|----------|------------|
| AWS Cloud Practitioner | AWS | ⭐ Beginner |
| AWS Solutions Architect | AWS | ⭐⭐ Intermediate |
| CKA (Kubernetes Admin) | CNCF | ⭐⭐⭐ Advanced |
| Terraform Associate | HashiCorp | ⭐⭐ Intermediate |
| Docker DCA | Docker | ⭐⭐ Intermediate |
| GCP Cloud Engineer | Google | ⭐⭐ Intermediate |
| Azure Administrator | Microsoft | ⭐⭐ Intermediate |

---

## 📚 Learning Resources

### Free Resources

| Topic | Resource |
|-------|----------|
| Linux | [Linux Journey](https://linuxjourney.com/) |
| Docker | [Docker Curriculum](https://docker-curriculum.com/) |
| Kubernetes | [K8s Tutorials](https://kubernetes.io/docs/tutorials/) |
| AWS | [AWS Free Tier](https://aws.amazon.com/free/) |
| Terraform | [Learn Terraform](https://learn.hashicorp.com/terraform) |
| CI/CD | [GitHub Actions Docs](https://docs.github.com/en/actions) |

### YouTube Channels

| Channel | Focus |
|---------|-------|
| TechWorld with Nana | DevOps tutorials |
| NetworkChuck | Networking & Linux |
| Fireship | Quick tech explainers |
| Traversy Media | Full stack & DevOps |
| KodeKloud | Kubernetes & DevOps |

### Books

| Book | Author |
|------|--------|
| The Phoenix Project | Gene Kim |
| The DevOps Handbook | Gene Kim |
| Site Reliability Engineering | Google |
| Kubernetes Up & Running | Kelsey Hightower |
| Docker Deep Dive | Nigel Poulton |

---

## 📋 Quick Reference Commands

### Service Management

| Action | Command |
|--------|---------|
| Start | `systemctl start SERVICE` |
| Stop | `systemctl stop SERVICE` |
| Restart | `systemctl restart SERVICE` |
| Status | `systemctl status SERVICE` |
| Enable | `systemctl enable SERVICE` |
| Logs | `journalctl -u SERVICE -f` |

### System Diagnostics

| Action | Command |
|--------|---------|
| Disk space | `df -h` |
| Memory | `free -h` |
| CPU | `htop` or `top` |
| Processes | `ps aux` |
| Open ports | `ss -tulpn` |
| Network | `netstat -an` |

### Docker

| Action | Command |
|--------|---------|
| List containers | `docker ps -a` |
| List images | `docker images` |
| Logs | `docker logs -f CONTAINER` |
| Enter container | `docker exec -it CONTAINER bash` |
| Cleanup | `docker system prune -a` |

### Kubernetes

| Action | Command |
|--------|---------|
| Get pods | `kubectl get pods` |
| Get services | `kubectl get svc` |
| Describe | `kubectl describe pod POD` |
| Logs | `kubectl logs POD` |
| Apply | `kubectl apply -f FILE.yaml` |

---

## 🔧 Troubleshooting Guide

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Cannot SSH | Check firewall: `ufw status`, check SSH: `systemctl status ssh` |
| 502 Bad Gateway | Check app: `pm2 status`, check Nginx: `nginx -t` |
| SSL errors | Check cert: `certbot certificates`, test renewal: `certbot renew --dry-run` |
| High memory | Check: `free -h`, find culprit: `ps aux --sort=-%mem \| head` |
| Disk full | Check: `df -h`, cleanup: `apt autoremove`, `docker system prune` |
| Port in use | Find: `ss -tulpn \| grep PORT`, kill: `kill -9 PID` |

---

## ✅ Final Checklist

### Server Security
- [ ] SSH key only (password disabled)
- [ ] Non-root user with sudo
- [ ] UFW firewall enabled
- [ ] Fail2Ban active
- [ ] SSL certificate installed
- [ ] Auto-updates enabled

### Application
- [ ] PM2/Docker running
- [ ] Environment variables secured
- [ ] Logging configured
- [ ] Health checks enabled
- [ ] Backups scheduled

### Monitoring
- [ ] Server metrics collected
- [ ] Application logs centralized
- [ ] Alerts configured
- [ ] Uptime monitoring active

---

> **🎉 Congratulations on completing this guide!**
>
> You now have a solid foundation in:
> - VPS setup and security
> - Nginx and SSL configuration
> - Docker and containerization
> - CI/CD pipelines
> - DevOps fundamentals
>
> Keep learning, keep building! 🚀

---

> **Created with ❤️ for Usama**  
> **Last Updated:** January 2026  
> **Guide Version:** 2.0