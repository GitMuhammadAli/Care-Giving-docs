# ⚖️ Load Balancing - Complete Guide

> A comprehensive guide to load balancing - Nginx, HAProxy, AWS ALB/NLB, algorithms, health checks, and high availability patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Load balancing distributes incoming network traffic across multiple servers to ensure high availability, reliability, and optimal resource utilization, using algorithms like round-robin, least connections, or IP hash to route requests while monitoring server health."

### The 7 Key Concepts (Remember These!)
```
1. LAYER 4 (L4)       → TCP/UDP level, fast, no content inspection
2. LAYER 7 (L7)       → HTTP level, content-based routing, SSL termination
3. ALGORITHM          → How to choose backend (round-robin, least-conn, etc.)
4. HEALTH CHECK       → Verify backends are alive and healthy
5. SESSION AFFINITY   → Sticky sessions - same client to same server
6. SSL TERMINATION    → LB handles TLS, backends receive plain HTTP
7. HIGH AVAILABILITY  → Multiple LBs with failover
```

### Load Balancer Types
```
┌─────────────────────────────────────────────────────────────────┐
│                  LOAD BALANCER TYPES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LAYER 4 (Transport Layer)                                     │
│  ─────────────────────────                                      │
│  • Operates on TCP/UDP                                         │
│  • No content inspection                                       │
│  • Very fast, low latency                                      │
│  • Example: AWS NLB, HAProxy (TCP mode)                        │
│                                                                 │
│  Client ──TCP──▶ [L4 LB] ──TCP──▶ Server                       │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  LAYER 7 (Application Layer)                                   │
│  ───────────────────────────                                    │
│  • Operates on HTTP/HTTPS                                      │
│  • Content-based routing (URL, headers)                        │
│  • SSL termination                                             │
│  • More features, slightly higher latency                      │
│  • Example: AWS ALB, Nginx, HAProxy (HTTP mode)                │
│                                                                 │
│  Client ──HTTPS──▶ [L7 LB] ──HTTP──▶ Server                    │
│                     ↓                                          │
│              SSL termination                                   │
│              Path routing                                      │
│              Header inspection                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Load Balancing Algorithms
```
┌─────────────────────────────────────────────────────────────────┐
│                  LOAD BALANCING ALGORITHMS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ROUND ROBIN                                                   │
│  ────────────                                                   │
│  Request 1 → Server A                                          │
│  Request 2 → Server B                                          │
│  Request 3 → Server C                                          │
│  Request 4 → Server A  (cycles back)                           │
│  • Simple, even distribution                                   │
│  • Doesn't consider server load                                │
│                                                                 │
│  WEIGHTED ROUND ROBIN                                          │
│  ────────────────────                                           │
│  Server A (weight 3): Gets 3x more requests                    │
│  Server B (weight 1): Gets 1x requests                         │
│  • For heterogeneous server capacities                         │
│                                                                 │
│  LEAST CONNECTIONS                                             │
│  ─────────────────                                              │
│  → Route to server with fewest active connections              │
│  • Better for varying request durations                        │
│  • More even load distribution                                 │
│                                                                 │
│  IP HASH                                                        │
│  ───────                                                        │
│  → hash(client_ip) → consistent server                         │
│  • Same client always hits same server                         │
│  • For session affinity without cookies                        │
│                                                                 │
│  LEAST RESPONSE TIME                                            │
│  ───────────────────                                            │
│  → Route to fastest responding server                          │
│  • Optimizes for user experience                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"SSL termination"** | "ALB handles SSL termination, backend traffic is HTTP" |
| **"Sticky sessions"** | "We use sticky sessions for stateful applications" |
| **"Health checks"** | "Health checks remove unhealthy backends from rotation" |
| **"Connection draining"** | "Connection draining gracefully removes servers from pool" |
| **"Blue-green with LB"** | "We switch traffic between blue/green via LB target groups" |
| **"Cross-zone balancing"** | "Cross-zone balancing distributes evenly across AZs" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Health check interval | **10-30 seconds** | Balance detection speed vs overhead |
| Unhealthy threshold | **2-3 failures** | Before removing from pool |
| Connection timeout | **60 seconds** | Default for most LBs |
| Idle timeout | **60 seconds** | Close idle connections |
| AWS ALB target limit | **1000** per target group | Scale consideration |

### The "Wow" Statement (Memorize This!)
> "We use AWS ALB for our microservices with path-based routing - /api/* goes to API target group, /web/* to frontend. ALB handles SSL termination with ACM certificates. Health checks verify /health endpoint every 10 seconds; two failures remove a target. We enable sticky sessions for stateful services using ALB-generated cookies with 1-hour duration. For high-throughput services, we use NLB for Layer 4 performance. Behind the LBs, we have auto-scaling groups that scale based on ALB request count per target metric. Connection draining ensures graceful deployments by completing in-flight requests before removing instances."

---

## 📚 Table of Contents

1. [Nginx Load Balancing](#1-nginx-load-balancing)
2. [HAProxy](#2-haproxy)
3. [AWS Load Balancers](#3-aws-load-balancers)
4. [Health Checks](#4-health-checks)
5. [SSL Termination](#5-ssl-termination)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Nginx Load Balancing

```nginx
# ════════════════════════════════════════════════════════════════
# NGINX LOAD BALANCER CONFIGURATION
# /etc/nginx/nginx.conf
# ════════════════════════════════════════════════════════════════

http {
    # Upstream block - define backend servers
    upstream api_servers {
        # Load balancing method (default: round-robin)
        # Options: least_conn, ip_hash, hash, random
        least_conn;
        
        # Backend servers with optional weight
        server 10.0.1.10:3000 weight=3;
        server 10.0.1.11:3000 weight=2;
        server 10.0.1.12:3000 weight=1;
        
        # Backup server (only used if all others are down)
        server 10.0.1.20:3000 backup;
        
        # Mark server as down (maintenance)
        server 10.0.1.13:3000 down;
        
        # Health check parameters
        server 10.0.1.10:3000 max_fails=3 fail_timeout=30s;
        
        # Keepalive connections to backends
        keepalive 32;
    }
    
    upstream websocket_servers {
        ip_hash;  # Sticky sessions for WebSocket
        server 10.0.2.10:8080;
        server 10.0.2.11:8080;
    }

    server {
        listen 80;
        listen 443 ssl http2;
        server_name api.example.com;
        
        # SSL configuration
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        
        # Proxy to upstream
        location /api/ {
            proxy_pass http://api_servers;
            
            # Proxy headers
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 5s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
            
            # Buffering
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 4k;
            
            # Connection reuse
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
        
        # WebSocket location
        location /ws/ {
            proxy_pass http://websocket_servers;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_read_timeout 3600s;  # Long timeout for WS
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}

# ════════════════════════════════════════════════════════════════
# NGINX PLUS - ACTIVE HEALTH CHECKS
# (Requires Nginx Plus subscription)
# ════════════════════════════════════════════════════════════════

upstream api_servers {
    zone api_servers 64k;
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    
    # Active health check (Nginx Plus only)
    health_check interval=5s fails=3 passes=2 uri=/health;
}
```

---

## 2. HAProxy

```haproxy
# ════════════════════════════════════════════════════════════════
# HAPROXY CONFIGURATION
# /etc/haproxy/haproxy.cfg
# ════════════════════════════════════════════════════════════════

global
    log /dev/log local0
    log /dev/log local1 notice
    maxconn 50000
    user haproxy
    group haproxy
    daemon
    
    # SSL settings
    ssl-default-bind-ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets
    tune.ssl.default-dh-param 2048

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    option  forwardfor
    option  http-server-close
    
    timeout connect 5s
    timeout client  30s
    timeout server  30s
    timeout http-request 10s
    timeout http-keep-alive 10s
    
    # Retry on connection failure
    retries 3
    option redispatch

# ════════════════════════════════════════════════════════════════
# FRONTEND - Receives incoming traffic
# ════════════════════════════════════════════════════════════════

frontend http_front
    bind *:80
    bind *:443 ssl crt /etc/haproxy/certs/example.com.pem
    
    # Redirect HTTP to HTTPS
    http-request redirect scheme https unless { ssl_fc }
    
    # Add security headers
    http-response set-header Strict-Transport-Security "max-age=31536000"
    
    # ACLs for routing
    acl is_api path_beg /api
    acl is_websocket hdr(Upgrade) -i websocket
    acl is_static path_beg /static
    
    # Route based on ACLs
    use_backend api_servers if is_api
    use_backend websocket_servers if is_websocket
    use_backend static_servers if is_static
    default_backend web_servers

# ════════════════════════════════════════════════════════════════
# BACKENDS - Server pools
# ════════════════════════════════════════════════════════════════

backend web_servers
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    
    # Sticky sessions using cookie
    cookie SERVERID insert indirect nocache
    
    server web1 10.0.1.10:3000 check cookie web1 weight 100
    server web2 10.0.1.11:3000 check cookie web2 weight 100
    server web3 10.0.1.12:3000 check cookie web3 weight 50

backend api_servers
    balance leastconn
    option httpchk GET /health
    http-check expect status 200
    
    # Rate limiting per backend
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request deny deny_status 429 if { sc_http_req_rate(0) gt 100 }
    
    server api1 10.0.2.10:3000 check
    server api2 10.0.2.11:3000 check
    server api3 10.0.2.12:3000 check backup

backend websocket_servers
    balance source  # IP-based sticky for WebSocket
    option httpchk GET /health
    
    timeout tunnel 3600s  # Long timeout for WebSocket
    
    server ws1 10.0.3.10:8080 check
    server ws2 10.0.3.11:8080 check

backend static_servers
    balance roundrobin
    
    # Cache control
    http-response set-header Cache-Control "public, max-age=31536000"
    
    server static1 10.0.4.10:80 check
    server static2 10.0.4.11:80 check

# ════════════════════════════════════════════════════════════════
# STATS PAGE
# ════════════════════════════════════════════════════════════════

listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
    stats auth admin:password
```

---

## 3. AWS Load Balancers

```hcl
# ════════════════════════════════════════════════════════════════
# AWS APPLICATION LOAD BALANCER (ALB) - Terraform
# ════════════════════════════════════════════════════════════════

# Application Load Balancer
resource "aws_lb" "main" {
  name               = "my-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = var.public_subnet_ids

  enable_deletion_protection = true
  enable_http2              = true
  idle_timeout              = 60

  access_logs {
    bucket  = aws_s3_bucket.lb_logs.bucket
    prefix  = "alb"
    enabled = true
  }

  tags = {
    Name        = "my-alb"
    Environment = var.environment
  }
}

# Target Group
resource "aws_lb_target_group" "api" {
  name        = "api-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"  # or "ip" for Fargate

  # Health check
  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 10
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    matcher             = "200-299"
  }

  # Stickiness
  stickiness {
    type            = "lb_cookie"
    cookie_duration = 3600
    enabled         = true
  }

  # Deregistration delay (connection draining)
  deregistration_delay = 30

  tags = {
    Name = "api-tg"
  }
}

# HTTPS Listener
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = aws_acm_certificate.main.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }
}

# HTTP to HTTPS redirect
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type = "redirect"
    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}

# Path-based routing
resource "aws_lb_listener_rule" "api" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }

  condition {
    path_pattern {
      values = ["/api/*"]
    }
  }
}

resource "aws_lb_listener_rule" "static" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 200

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.static.arn
  }

  condition {
    path_pattern {
      values = ["/static/*", "/assets/*"]
    }
  }
}

# ════════════════════════════════════════════════════════════════
# AWS NETWORK LOAD BALANCER (NLB)
# ════════════════════════════════════════════════════════════════

resource "aws_lb" "nlb" {
  name               = "my-nlb"
  internal           = false
  load_balancer_type = "network"
  subnets            = var.public_subnet_ids

  enable_cross_zone_load_balancing = true

  tags = {
    Name = "my-nlb"
  }
}

resource "aws_lb_target_group" "tcp" {
  name        = "tcp-tg"
  port        = 3000
  protocol    = "TCP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 2
    interval            = 10
    protocol            = "TCP"
  }

  # Preserve client IP
  preserve_client_ip = true
}

resource "aws_lb_listener" "tcp" {
  load_balancer_arn = aws_lb.nlb.arn
  port              = 443
  protocol          = "TLS"
  certificate_arn   = aws_acm_certificate.main.arn
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tcp.arn
  }
}
```

---

## 4. Health Checks

```yaml
# ════════════════════════════════════════════════════════════════
# HEALTH CHECK PATTERNS
# ════════════════════════════════════════════════════════════════

# Application health endpoint (Node.js example)
```

```typescript
// Express.js health check endpoint
import express from 'express';
import { Pool } from 'pg';
import Redis from 'ioredis';

const app = express();
const db = new Pool({ connectionString: process.env.DATABASE_URL });
const redis = new Redis(process.env.REDIS_URL);

// Simple liveness check
app.get('/health/live', (req, res) => {
  res.status(200).json({ status: 'alive' });
});

// Readiness check (with dependencies)
app.get('/health/ready', async (req, res) => {
  const checks = {
    database: false,
    redis: false,
    memory: false
  };

  try {
    // Database check
    await db.query('SELECT 1');
    checks.database = true;
  } catch (e) {
    console.error('Database health check failed:', e);
  }

  try {
    // Redis check
    await redis.ping();
    checks.redis = true;
  } catch (e) {
    console.error('Redis health check failed:', e);
  }

  // Memory check
  const used = process.memoryUsage();
  const heapUsedPercent = (used.heapUsed / used.heapTotal) * 100;
  checks.memory = heapUsedPercent < 90;

  const allHealthy = Object.values(checks).every(v => v);
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ready' : 'not ready',
    checks,
    uptime: process.uptime()
  });
});

// Deep health check (for debugging, not for LB)
app.get('/health/deep', async (req, res) => {
  // More detailed checks...
});
```

---

## 5. SSL Termination

```nginx
# ════════════════════════════════════════════════════════════════
# SSL TERMINATION - NGINX
# ════════════════════════════════════════════════════════════════

server {
    listen 443 ssl http2;
    server_name example.com;

    # SSL certificates
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # SSL session caching
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;

    location / {
        proxy_pass http://backend_servers;
        
        # Pass original protocol to backend
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

---

## 6. Common Pitfalls

```nginx
# ════════════════════════════════════════════════════════════════
# LOAD BALANCING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: No health checks
# Bad - LB sends traffic to dead servers
upstream backend {
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
}

# Good - Health checks enabled
upstream backend {
    server 10.0.1.10:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3000 max_fails=3 fail_timeout=30s;
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 2: Wrong algorithm for use case
# Bad - Round robin for stateful sessions
upstream backend {
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
}

# Good - IP hash for session affinity
upstream backend {
    ip_hash;
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 3: No connection draining
# Requests fail during deployment

# Good - Connection draining (AWS ALB)
# deregistration_delay = 30  # Allows 30s to complete requests

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 4: Missing X-Forwarded headers
# Bad - Backend sees LB IP, not client IP
location / {
    proxy_pass http://backend;
}

# Good - Forward real client IP
location / {
    proxy_pass http://backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 5: Timeout mismatch
# Bad - LB timeout shorter than app processing time
proxy_read_timeout 30s;  # But API takes 60s

# Good - Match timeouts to app behavior
proxy_read_timeout 90s;  # Longer than max expected response
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is a load balancer?"**
> "Distributes incoming traffic across multiple servers for high availability, scalability, and reliability. Routes requests using algorithms like round-robin, least connections. Performs health checks to avoid sending traffic to unhealthy servers."

**Q: "Layer 4 vs Layer 7 load balancing?"**
> "L4 operates at TCP/UDP level - fast, simple, no content inspection. L7 operates at HTTP level - can route based on URL, headers, cookies, and perform SSL termination. Use L4 for pure performance, L7 for content-based routing."

**Q: "What algorithms do load balancers use?"**
> "Round-robin: equal distribution in rotation. Weighted round-robin: proportion based on server capacity. Least connections: route to server with fewest active connections. IP hash: consistent server per client IP. Least response time: route to fastest server."

### Intermediate Questions

**Q: "How do sticky sessions work?"**
> "Ensures requests from same client go to same server. Methods: cookie-based (LB sets cookie with server ID), IP hash (hash client IP to server), or application-managed session ID. Needed for stateful apps but reduces load distribution effectiveness."

**Q: "What is SSL termination?"**
> "Load balancer handles TLS encryption/decryption. Clients connect via HTTPS to LB, LB forwards plain HTTP to backends. Benefits: offloads CPU from backends, centralized certificate management. Backend traffic should be on private network."

**Q: "How do health checks work?"**
> "LB periodically probes backends (HTTP GET, TCP connect). If probe fails X times (unhealthy threshold), server removed from rotation. Once healthy again (healthy threshold passes), re-added. Protects users from hitting dead servers."

### Advanced Questions

**Q: "How would you design a highly available load balancer setup?"**
> "Multiple LBs in active-active or active-passive. DNS-based failover or floating IP (keepalived). Cross-zone load balancing for AZ failures. Auto-scaling LB capacity or managed service (ALB). Health checks at multiple layers. Geographic distribution with global load balancing (Route53, CloudFront)."

**Q: "How do you handle WebSocket connections with load balancers?"**
> "Use sticky sessions (IP hash or session cookie) for persistent connections. Set long timeouts (hours). Use connection upgrade headers properly. Consider dedicated LB for WebSocket traffic. Monitor connection counts per server."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  LOAD BALANCER COMPARISON                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AWS ALB (Application)                                         │
│  • Layer 7 (HTTP/HTTPS)                                        │
│  • Path/host-based routing                                     │
│  • WebSocket support                                           │
│  • SSL termination                                             │
│                                                                 │
│  AWS NLB (Network)                                             │
│  • Layer 4 (TCP/UDP)                                           │
│  • Ultra-low latency                                           │
│  • Static IP support                                           │
│  • Millions of requests/sec                                    │
│                                                                 │
│  Nginx                                                          │
│  • Layer 7                                                     │
│  • Highly configurable                                         │
│  • Free (Plus for advanced)                                    │
│                                                                 │
│  HAProxy                                                        │
│  • Layer 4 & 7                                                 │
│  • Very high performance                                       │
│  • Advanced ACLs                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

ALGORITHMS:
┌────────────────────────────────────────────────────────────────┐
│ Round Robin       - Simple rotation                            │
│ Least Connections - Fewest active connections                  │
│ IP Hash           - Consistent server per client               │
│ Weighted          - Based on server capacity                   │
│ Least Time        - Fastest response (Nginx Plus)              │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

