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