# Chapter 07: Networking Deep Dive

> "Understanding networking is understanding how the internet works."

---

## 🌐 The OSI Model

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 7: Application   │ HTTP, FTP, SMTP, DNS                   │
│         (Data)         │ What the user interacts with           │
├─────────────────────────────────────────────────────────────────┤
│ Layer 6: Presentation  │ SSL/TLS, JPEG, ASCII                   │
│         (Data)         │ Encryption, compression, encoding      │
├─────────────────────────────────────────────────────────────────┤
│ Layer 5: Session       │ NetBIOS, RPC                           │
│         (Data)         │ Managing sessions, connections         │
├─────────────────────────────────────────────────────────────────┤
│ Layer 4: Transport     │ TCP, UDP                               │
│       (Segments)       │ Reliable delivery, ports               │
├─────────────────────────────────────────────────────────────────┤
│ Layer 3: Network       │ IP, ICMP, ARP                          │
│       (Packets)        │ Routing, addressing                    │
├─────────────────────────────────────────────────────────────────┤
│ Layer 2: Data Link     │ Ethernet, WiFi, MAC                    │
│        (Frames)        │ Node-to-node delivery                  │
├─────────────────────────────────────────────────────────────────┤
│ Layer 1: Physical      │ Cables, radio waves, fiber             │
│         (Bits)         │ Physical transmission                  │
└─────────────────────────────────────────────────────────────────┘
```

### Simplified TCP/IP Model

```
┌───────────────────────────────────────────────────┐
│ Application    │ HTTP, DNS, SSH, FTP             │
├───────────────────────────────────────────────────┤
│ Transport      │ TCP, UDP                         │
├───────────────────────────────────────────────────┤
│ Internet       │ IP                               │
├───────────────────────────────────────────────────┤
│ Network Access │ Ethernet, WiFi                   │
└───────────────────────────────────────────────────┘
```

---

## 📡 TCP/IP Deep Dive

### IP Addressing

```
IPv4: 192.168.1.1 (32 bits)
┌──────────┬──────────┬──────────┬──────────┐
│    192   │    168   │     1    │     1    │
│ 11000000 │ 10101000 │ 00000001 │ 00000001 │
└──────────┴──────────┴──────────┴──────────┘

IPv6: 2001:0db8:85a3:0000:0000:8a2e:0370:7334 (128 bits)

Private IP ranges:
10.0.0.0/8       (10.0.0.0 - 10.255.255.255)
172.16.0.0/12    (172.16.0.0 - 172.31.255.255)
192.168.0.0/16   (192.168.0.0 - 192.168.255.255)

CIDR Notation:
192.168.1.0/24 = 256 addresses (192.168.1.0 - 192.168.1.255)
192.168.1.0/16 = 65,536 addresses
10.0.0.0/8     = 16,777,216 addresses
```

### TCP Three-Way Handshake

```
Client                              Server
   │                                   │
   │ ─────── SYN (seq=x) ────────────► │
   │         "Hey, want to talk?"      │
   │                                   │
   │ ◄─── SYN-ACK (seq=y, ack=x+1) ─── │
   │      "Sure, I'm ready too"        │
   │                                   │
   │ ─────── ACK (ack=y+1) ──────────► │
   │         "Great, let's go!"        │
   │                                   │
   │ ◄═══════ DATA TRANSFER ═════════► │
   │                                   │
   │ ─────── FIN ────────────────────► │
   │ ◄────── ACK ──────────────────── │
   │ ◄────── FIN ──────────────────── │
   │ ─────── ACK ────────────────────► │
   │                                   │
```

### TCP vs UDP

```
┌─────────────────────────────────────────────────────────────────┐
│                           TCP                                   │
├─────────────────────────────────────────────────────────────────┤
│ - Connection-oriented (handshake)                               │
│ - Reliable (acknowledgments, retransmission)                    │
│ - Ordered delivery (sequence numbers)                           │
│ - Flow control (don't overwhelm receiver)                       │
│ - Congestion control (don't overwhelm network)                  │
│ - Slower, more overhead                                         │
│                                                                 │
│ Use for: HTTP, email, file transfer, SSH                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                           UDP                                   │
├─────────────────────────────────────────────────────────────────┤
│ - Connectionless (no handshake)                                 │
│ - Unreliable (no acknowledgments)                               │
│ - No ordering guarantee                                         │
│ - No flow/congestion control                                    │
│ - Fast, minimal overhead                                        │
│                                                                 │
│ Use for: DNS, video streaming, gaming, VoIP                     │
└─────────────────────────────────────────────────────────────────┘
```

### Ports

```
Well-known ports (0-1023):
┌────────┬─────────────────────────────────────┐
│ Port   │ Service                             │
├────────┼─────────────────────────────────────┤
│ 20, 21 │ FTP (data, control)                 │
│ 22     │ SSH                                 │
│ 23     │ Telnet                              │
│ 25     │ SMTP (email sending)                │
│ 53     │ DNS                                 │
│ 80     │ HTTP                                │
│ 110    │ POP3 (email receiving)              │
│ 143    │ IMAP (email)                        │
│ 443    │ HTTPS                               │
│ 3306   │ MySQL                               │
│ 5432   │ PostgreSQL                          │
│ 6379   │ Redis                               │
│ 27017  │ MongoDB                             │
└────────┴─────────────────────────────────────┘

Ephemeral ports (49152-65535):
Used by clients for outgoing connections
```

---

## 🔐 TLS/SSL Explained

### What SSL/TLS Does

```
1. Encryption: Data can't be read by eavesdroppers
2. Authentication: Server proves its identity
3. Integrity: Data can't be tampered with

HTTP (insecure):
Client ──────── "password123" ──────────► Server
         └── Anyone can read this!

HTTPS (TLS):
Client ──────── "x7$#@kL9m..." ──────────► Server
         └── Encrypted, unreadable
```

### TLS Handshake

```
Client                                  Server
   │                                       │
   │ ──── ClientHello ────────────────────►│
   │      (Supported TLS versions,         │
   │       cipher suites, random)          │
   │                                       │
   │ ◄─── ServerHello ─────────────────────│
   │      (Chosen TLS version,             │
   │       cipher suite, random)           │
   │                                       │
   │ ◄─── Certificate ─────────────────────│
   │      (Server's public key)            │
   │                                       │
   │ ◄─── ServerHelloDone ─────────────────│
   │                                       │
   │ ──── ClientKeyExchange ──────────────►│
   │      (Encrypted pre-master secret)    │
   │                                       │
   │ ──── ChangeCipherSpec ───────────────►│
   │ ──── Finished ───────────────────────►│
   │                                       │
   │ ◄─── ChangeCipherSpec ────────────────│
   │ ◄─── Finished ────────────────────────│
   │                                       │
   │ ◄═══ Encrypted Communication ════════►│
```

### Certificate Chain

```
┌───────────────────────────────────────────────────────────────┐
│                    Root CA Certificate                        │
│              (Trusted by browsers/OS)                         │
│                   Verisign, DigiCert                          │
└───────────────────────────┬───────────────────────────────────┘
                            │ Signs
                            ▼
┌───────────────────────────────────────────────────────────────┐
│                 Intermediate CA Certificate                   │
│                    (Validates domains)                        │
└───────────────────────────┬───────────────────────────────────┘
                            │ Signs
                            ▼
┌───────────────────────────────────────────────────────────────┐
│                   Your Server Certificate                     │
│                     example.com                               │
└───────────────────────────────────────────────────────────────┘
```

### mTLS (Mutual TLS)

```
Regular TLS:
- Client verifies server's certificate
- Server doesn't verify client

mTLS (Mutual TLS):
- Client verifies server's certificate
- Server verifies client's certificate
- Both parties authenticated

Used for: Service-to-service communication, zero trust
```

---

## 🔑 SSH Explained

### How SSH Works

```
1. Connection established
2. Key exchange (Diffie-Hellman)
3. Server authentication (host key)
4. User authentication (password or key)
5. Encrypted session

SSH Key Types:
- RSA (most common, 2048+ bits recommended)
- Ed25519 (newer, smaller, faster)
- ECDSA (elliptic curve)
```

### SSH Key Authentication

```
┌──────────────────────────────────────────────────────────────┐
│                     Your Computer                            │
│  ~/.ssh/id_rsa       (Private key - NEVER share!)            │
│  ~/.ssh/id_rsa.pub   (Public key - safe to share)            │
└──────────────────────────────────────────────────────────────┘
                              │
                              │ Copy public key
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                        Server                                │
│  ~/.ssh/authorized_keys   (Contains your public key)         │
└──────────────────────────────────────────────────────────────┘

Authentication:
1. Client says "I'm user X"
2. Server sends random challenge
3. Client signs challenge with private key
4. Server verifies with public key
5. Access granted (private key never transmitted)
```

### SSH Commands

```bash
# Generate key pair
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to server
ssh-copy-id user@server

# Connect
ssh user@server

# SSH tunnel (local port forwarding)
ssh -L 8080:localhost:80 user@server
# Now localhost:8080 → server:80

# SSH tunnel (remote port forwarding)
ssh -R 8080:localhost:3000 user@server
# Now server:8080 → your localhost:3000

# SSH config (~/.ssh/config)
Host myserver
    HostName 192.168.1.100
    User admin
    IdentityFile ~/.ssh/id_ed25519
    Port 22

# Now just: ssh myserver
```

---

## 🌍 DNS Explained

### How DNS Works

```
You type: www.google.com

1. Browser cache check
2. OS cache check
3. Router cache check
4. ISP DNS resolver
   │
   ├─► Root DNS Server (.)
   │   "I know who handles .com"
   │
   ├─► TLD DNS Server (.com)
   │   "I know who handles google.com"
   │
   └─► Authoritative DNS Server (google.com)
       "www.google.com = 142.250.185.68"
       
5. Response cached at each level
6. Browser connects to 142.250.185.68
```

### DNS Record Types

```
A Record:       Domain → IPv4 address
                example.com → 93.184.216.34

AAAA Record:    Domain → IPv6 address
                example.com → 2606:2800:220:1:248:1893:25c8:1946

CNAME Record:   Alias → Another domain
                www.example.com → example.com

MX Record:      Mail server for domain
                example.com → mail.example.com (priority 10)

TXT Record:     Arbitrary text (verification, SPF, DKIM)
                example.com → "v=spf1 include:_spf.google.com ~all"

NS Record:      Nameserver for domain
                example.com → ns1.example.com

SOA Record:     Start of Authority (primary nameserver info)

PTR Record:     Reverse DNS (IP → Domain)
                34.216.184.93 → example.com
```

### DNS TTL

```
TTL (Time To Live):
- How long to cache the record
- Lower TTL = faster propagation, more DNS queries
- Higher TTL = slower propagation, less queries

Common values:
- 300 (5 min): Dynamic records, during migrations
- 3600 (1 hour): Normal records
- 86400 (1 day): Stable records
```

---

## 🌐 HTTP Deep Dive

### HTTP/1.1 vs HTTP/2 vs HTTP/3

```
HTTP/1.1:
┌────────────────────────────────────────────────────────────┐
│ - One request per connection at a time                     │
│ - Head-of-line blocking                                    │
│ - Text-based headers (verbose)                             │
│ - Multiple TCP connections needed for parallelism          │
└────────────────────────────────────────────────────────────┘

HTTP/2:
┌────────────────────────────────────────────────────────────┐
│ - Multiplexing (multiple requests on one connection)       │
│ - Binary framing (efficient)                               │
│ - Header compression (HPACK)                               │
│ - Server push                                              │
│ - Stream prioritization                                    │
│ - Still TCP (head-of-line blocking at transport layer)     │
└────────────────────────────────────────────────────────────┘

HTTP/3:
┌────────────────────────────────────────────────────────────┐
│ - Based on QUIC (UDP-based)                                │
│ - No head-of-line blocking                                 │
│ - Faster connection setup (0-RTT)                          │
│ - Built-in encryption                                      │
│ - Connection migration (WiFi → cellular)                   │
└────────────────────────────────────────────────────────────┘
```

### HTTP Methods

```
GET     - Retrieve resource (idempotent, cacheable)
POST    - Create resource (not idempotent)
PUT     - Replace resource (idempotent)
PATCH   - Partial update (not idempotent)
DELETE  - Remove resource (idempotent)
HEAD    - GET without body (check if exists)
OPTIONS - Get allowed methods (CORS preflight)
```

### HTTP Status Codes

```
1xx - Informational
  100 Continue
  101 Switching Protocols

2xx - Success
  200 OK
  201 Created
  204 No Content
  
3xx - Redirection
  301 Moved Permanently
  302 Found (temporary redirect)
  304 Not Modified (cache valid)
  
4xx - Client Error
  400 Bad Request
  401 Unauthorized (not authenticated)
  403 Forbidden (not authorized)
  404 Not Found
  405 Method Not Allowed
  409 Conflict
  422 Unprocessable Entity
  429 Too Many Requests
  
5xx - Server Error
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable
  504 Gateway Timeout
```

### Important HTTP Headers

```http
# Request Headers
Host: example.com
User-Agent: Mozilla/5.0...
Accept: application/json
Accept-Encoding: gzip, deflate
Authorization: Bearer <token>
Cookie: session=abc123
Content-Type: application/json
Origin: https://example.com
Referer: https://example.com/page

# Response Headers
Content-Type: application/json; charset=utf-8
Content-Length: 1234
Content-Encoding: gzip
Cache-Control: max-age=3600
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
Set-Cookie: session=abc123; HttpOnly; Secure
Access-Control-Allow-Origin: *
Strict-Transport-Security: max-age=31536000

# Security Headers
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

---

## 🔌 WebSockets

### WebSocket Handshake

```
Client → Server (HTTP Upgrade Request):
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

Server → Client (Upgrade Response):
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

After handshake: Full-duplex communication
```

### WebSocket vs HTTP

```
HTTP:
Client ──request──► Server
Client ◄──response── Server
(Connection closed or kept alive for next request)

WebSocket:
Client ◄══════════► Server
(Persistent, bidirectional connection)
Both can send messages anytime
```

### When to Use WebSockets

```
Use WebSockets for:
- Real-time chat
- Live notifications
- Gaming
- Collaborative editing
- Live dashboards
- Stock tickers

Use HTTP/SSE for:
- One-way server updates
- When WebSocket isn't available
- Simpler implementation needed
```

---

## 📖 Further Reading

- "Computer Networking: A Top-Down Approach"
- "TCP/IP Illustrated" by Stevens
- Cloudflare Learning Center
- Julia Evans' networking zines

---

**Next:** [Chapter 08: Security Engineering →](./08-security-engineering.md)


