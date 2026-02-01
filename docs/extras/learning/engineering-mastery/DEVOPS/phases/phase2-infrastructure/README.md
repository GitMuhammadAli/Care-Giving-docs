# Phase 2: Cloud & Infrastructure
## Weeks 4-6 | Your First Real Server

> **Prerequisites:** Completed Phase 1 (Linux, Bash, Git)  
> **Time:** 10-15 hours per week  
> **Outcome:** Deploy an application to a real server with HTTPS

---

## 🎯 Learning Objectives

By the end of Phase 2, you will:
- [ ] Understand how the internet works (DNS, HTTP, ports)
- [ ] Configure firewalls and network security
- [ ] Set up Nginx as a web server and reverse proxy
- [ ] Obtain and configure SSL certificates (HTTPS)
- [ ] Deploy a full application to a VPS

---

## 📅 Weekly Breakdown

### Week 4: Networking & Web Fundamentals

**What you'll learn:**
- How the internet works (IP, DNS, HTTP)
- Domain names and DNS records
- Ports and protocols
- Firewalls and security groups

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [Deployment-Fundamentals-Guide.md](../../Deployment-Fundamentals-Guide.md) | Chapters 1-2: Internet & DNS | 2 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 4: Networking Fundamentals | 2 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 5: Firewall & Network Security | 1 hour |

**Key concepts:**

```
HTTP Request Journey:
┌──────────┐     ┌─────────┐     ┌───────────┐     ┌──────────┐
│  Browser │────▶│   DNS   │────▶│  Firewall │────▶│  Server  │
│          │     │         │     │           │     │          │
│ google.com────▶│142.250.x│────▶│ Port 443? │────▶│  Nginx   │
└──────────┘     └─────────┘     └───────────┘     └──────────┘
```

**Hands-on exercises:**
```bash
# Exercise 1: DNS lookup
nslookup google.com
dig google.com

# Exercise 2: Check open ports
sudo netstat -tulpn
ss -tulpn

# Exercise 3: Test connectivity
ping google.com
curl -I https://google.com
telnet google.com 443

# Exercise 4: Configure UFW firewall
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

**Important ports to know:**
| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | SSH | Remote access |
| 80 | HTTP | Web traffic (unencrypted) |
| 443 | HTTPS | Web traffic (encrypted) |
| 3000 | - | Node.js development |
| 5432 | PostgreSQL | Database |
| 6379 | Redis | Cache |

**Checkpoint quiz:**
1. What does DNS do?
2. Why should you block port 5432 from the internet?
3. What's the difference between port 80 and 443?

---

### Week 5: Web Servers (Nginx)

**What you'll learn:**
- What is a web server
- Nginx configuration
- Reverse proxy setup
- SSL/TLS and HTTPS
- Security headers

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [Deployment-Fundamentals-Guide.md](../../Deployment-Fundamentals-Guide.md) | Chapters 3-6: Web Servers & SSL | 3 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 6: Web Servers (Nginx) | 2 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 11: SSL/TLS & HTTPS | 1 hour |

**Key concepts:**

```
Reverse Proxy Pattern:
┌──────────┐     ┌───────────┐     ┌──────────────┐
│  Client  │────▶│   Nginx   │────▶│  Node.js     │
│          │     │  :80/443  │     │  :3000       │
│ HTTPS    │     │  (public) │     │  (internal)  │
└──────────┘     └───────────┘     └──────────────┘
                      │
                 Handles:
                 - SSL/HTTPS
                 - Caching
                 - Compression
                 - Load balancing
```

**Hands-on exercises:**

```bash
# Exercise 1: Install Nginx
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Visit http://localhost - you should see "Welcome to nginx!"
```

```nginx
# Exercise 2: Basic server block
# /etc/nginx/sites-available/myapp

server {
    listen 80;
    server_name myapp.local;
    
    location / {
        root /var/www/myapp;
        index index.html;
    }
}
```

```nginx
# Exercise 3: Reverse proxy configuration
# /etc/nginx/sites-available/api

server {
    listen 80;
    server_name api.myapp.local;
    
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Exercise 4: Enable site and test
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t          # Test configuration
sudo systemctl reload nginx
```

**Nginx commands cheatsheet:**
```bash
sudo nginx -t                    # Test config
sudo systemctl reload nginx      # Apply changes (no downtime)
sudo systemctl restart nginx     # Restart
sudo tail -f /var/log/nginx/error.log    # View errors
sudo tail -f /var/log/nginx/access.log   # View access logs
```

**Checkpoint quiz:**
1. What is a reverse proxy?
2. Why use Nginx in front of Node.js?
3. What does `proxy_pass` do?

---

### Week 6: Full VPS Deployment

**What you'll learn:**
- Get a free VPS (Oracle Cloud)
- SSH key setup and secure access
- Full stack deployment
- SSL certificate with Let's Encrypt
- PM2 process manager

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [Complete-vps-setup-guide.md](../../Complete-vps-setup-guide.md) | All phases | 4 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 7: Process Management (PM2) | 1 hour |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 14: Security Hardening | 1 hour |

**Step-by-step deployment:**

```bash
# Step 1: Get a VPS
# Sign up for Oracle Cloud Free Tier
# Create an Ubuntu 22.04 instance
# Download your SSH private key

# Step 2: Connect via SSH
ssh -i ~/path/to/key.pem ubuntu@your-server-ip

# Step 3: Initial server setup
sudo apt update && sudo apt upgrade -y
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Step 4: Install Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Step 5: Install PM2
sudo npm install -g pm2

# Step 6: Clone your app
git clone https://github.com/yourusername/yourapp.git
cd yourapp
npm install
npm run build

# Step 7: Start with PM2
pm2 start npm --name "myapp" -- start
pm2 save
pm2 startup  # Follow the output instructions

# Step 8: Configure Nginx reverse proxy
sudo nano /etc/nginx/sites-available/myapp
# (Add reverse proxy config from Week 5)
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Step 9: Get SSL certificate
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com

# Step 10: Verify
curl https://yourdomain.com
```

**SSL with Let's Encrypt:**
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate (auto-configures Nginx)
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Test auto-renewal
sudo certbot renew --dry-run

# Certificates auto-renew via cron
```

**PM2 commands cheatsheet:**
```bash
pm2 start app.js           # Start app
pm2 start npm -- start     # Start npm script
pm2 list                   # List all processes
pm2 logs                   # View logs
pm2 restart app            # Restart
pm2 stop app               # Stop
pm2 delete app             # Remove from PM2
pm2 monit                  # Live monitoring
pm2 save                   # Save process list
pm2 startup                # Configure auto-start
```

**Checkpoint quiz:**
1. Why disable password authentication for SSH?
2. What does PM2 provide that `node app.js` doesn't?
3. How often do Let's Encrypt certificates need renewal?

---

## 📋 Phase 2 Project: Deploy to Production

**Deploy a Node.js application to a VPS with:**
1. SSH key authentication (no passwords)
2. Firewall configured (only 22, 80, 443 open)
3. Nginx as reverse proxy
4. HTTPS with Let's Encrypt
5. PM2 for process management
6. Auto-start on server reboot

**Deployment checklist:**
```
[ ] VPS created and accessible via SSH
[ ] System updated (apt update/upgrade)
[ ] Firewall configured (UFW)
[ ] Node.js installed
[ ] Application cloned and built
[ ] PM2 running application
[ ] PM2 startup configured
[ ] Nginx reverse proxy configured
[ ] SSL certificate obtained
[ ] Domain pointing to server
[ ] HTTPS working
[ ] Application accessible publicly
```

---

## ✅ Phase 2 Completion Checklist

Before moving to Phase 3, ensure you can:

- [ ] Explain how DNS resolves a domain to an IP
- [ ] Configure a firewall with UFW
- [ ] Set up Nginx as a reverse proxy
- [ ] Obtain SSL certificates with Certbot
- [ ] Deploy an app to a VPS with HTTPS
- [ ] Manage processes with PM2
- [ ] Complete the Phase 2 project (full deployment)

---

## 🔗 Quick Reference

**Nginx config template:**
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    
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

---

## ➡️ Next Step

Ready for Phase 3? [Infrastructure as Code →](../phase3-iac/README.md)

