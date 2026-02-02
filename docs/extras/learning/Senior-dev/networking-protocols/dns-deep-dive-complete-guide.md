# DNS Deep Dive - Complete Guide

> **MUST REMEMBER**: DNS translates domain names to IP addresses through a hierarchical system (root → TLD → authoritative). Key concepts: TTL controls caching duration, A/AAAA records for IPs, CNAME for aliases, MX for email. DNS-based load balancing uses multiple A records or GeoDNS. Common issues: propagation delays (TTL-dependent), misconfigured records, DNSSEC validation failures.

---

## How to Explain Like a Senior Developer

"DNS is the internet's phone book - it maps human-readable names to IP addresses. When you hit example.com, your browser asks a resolver which recursively queries: root servers (who knows .com?), TLD servers (who knows example.com?), and finally the authoritative nameserver (what's the IP?). Results are cached based on TTL - set it low before changes, high for stability. For load balancing, you can return multiple A records (round-robin), use weighted records, or GeoDNS to route users to nearest servers. DNS is also critical for email (MX records), service discovery (SRV), and verification (TXT for SPF/DKIM). Understanding DNS is essential because it's often the silent cause of outages."

---

## Core Implementation

### DNS Resolution Flow

```typescript
// dns/resolution.ts
import * as dns from 'dns';
import { promisify } from 'util';

const resolve4 = promisify(dns.resolve4);
const resolve6 = promisify(dns.resolve6);
const resolveMx = promisify(dns.resolveMx);
const resolveTxt = promisify(dns.resolveTxt);
const resolveCname = promisify(dns.resolveCname);
const resolveSrv = promisify(dns.resolveSrv);
const resolveNs = promisify(dns.resolveNs);
const reverse = promisify(dns.reverse);

interface DnsInfo {
  domain: string;
  ipv4: string[];
  ipv6: string[];
  mx: Array<{ exchange: string; priority: number }>;
  txt: string[][];
  cname: string[];
  ns: string[];
  srv: Array<{ name: string; port: number; priority: number; weight: number }>;
}

// Comprehensive DNS lookup
export async function getDnsInfo(domain: string): Promise<DnsInfo> {
  const result: DnsInfo = {
    domain,
    ipv4: [],
    ipv6: [],
    mx: [],
    txt: [],
    cname: [],
    ns: [],
    srv: [],
  };
  
  // Resolve all record types in parallel
  const [ipv4, ipv6, mx, txt, cname, ns] = await Promise.allSettled([
    resolve4(domain),
    resolve6(domain),
    resolveMx(domain),
    resolveTxt(domain),
    resolveCname(domain),
    resolveNs(domain),
  ]);
  
  if (ipv4.status === 'fulfilled') result.ipv4 = ipv4.value;
  if (ipv6.status === 'fulfilled') result.ipv6 = ipv6.value;
  if (mx.status === 'fulfilled') result.mx = mx.value;
  if (txt.status === 'fulfilled') result.txt = txt.value;
  if (cname.status === 'fulfilled') result.cname = cname.value;
  if (ns.status === 'fulfilled') result.ns = ns.value;
  
  return result;
}

// Reverse DNS lookup (IP to domain)
export async function reverseLookup(ip: string): Promise<string[]> {
  try {
    return await reverse(ip);
  } catch (error) {
    return [];
  }
}

// Check DNS propagation across multiple resolvers
export async function checkPropagation(
  domain: string,
  expectedIp: string
): Promise<{ resolver: string; ip: string; matches: boolean }[]> {
  const publicResolvers = [
    { name: 'Google', ip: '8.8.8.8' },
    { name: 'Cloudflare', ip: '1.1.1.1' },
    { name: 'OpenDNS', ip: '208.67.222.222' },
    { name: 'Quad9', ip: '9.9.9.9' },
  ];
  
  const results = await Promise.all(
    publicResolvers.map(async (resolver) => {
      const customResolver = new dns.Resolver();
      customResolver.setServers([resolver.ip]);
      
      try {
        const addresses = await promisify(
          customResolver.resolve4.bind(customResolver)
        )(domain);
        
        return {
          resolver: resolver.name,
          ip: addresses[0],
          matches: addresses.includes(expectedIp),
        };
      } catch (error) {
        return {
          resolver: resolver.name,
          ip: 'FAILED',
          matches: false,
        };
      }
    })
  );
  
  return results;
}

// Usage
async function main() {
  const info = await getDnsInfo('google.com');
  console.log('DNS Info:', info);
  
  const propagation = await checkPropagation('example.com', '93.184.216.34');
  console.log('Propagation:', propagation);
}
```

### DNS Record Types Explained

```typescript
// dns/record-types.ts

/**
 * DNS Record Types and Their Uses
 */

interface DnsRecords {
  // A Record - Maps domain to IPv4 address
  A: {
    example: 'example.com. 300 IN A 93.184.216.34';
    use: 'Primary domain to IP mapping';
  };
  
  // AAAA Record - Maps domain to IPv6 address
  AAAA: {
    example: 'example.com. 300 IN AAAA 2606:2800:220:1:248:1893:25c8:1946';
    use: 'IPv6 domain mapping';
  };
  
  // CNAME Record - Alias to another domain
  CNAME: {
    example: 'www.example.com. 300 IN CNAME example.com.';
    use: 'Alias one domain to another (cannot coexist with other records)';
  };
  
  // MX Record - Mail server
  MX: {
    example: 'example.com. 300 IN MX 10 mail.example.com.';
    use: 'Email routing (priority number - lower = higher priority)';
  };
  
  // TXT Record - Text data
  TXT: {
    example: 'example.com. 300 IN TXT "v=spf1 include:_spf.google.com ~all"';
    use: 'SPF, DKIM, domain verification, arbitrary data';
  };
  
  // NS Record - Nameserver
  NS: {
    example: 'example.com. 86400 IN NS ns1.example.com.';
    use: 'Delegates DNS authority to nameservers';
  };
  
  // SRV Record - Service location
  SRV: {
    example: '_http._tcp.example.com. 300 IN SRV 10 5 80 www.example.com.';
    use: 'Service discovery (priority, weight, port, target)';
  };
  
  // CAA Record - Certificate Authority Authorization
  CAA: {
    example: 'example.com. 300 IN CAA 0 issue "letsencrypt.org"';
    use: 'Restrict which CAs can issue certificates';
  };
  
  // PTR Record - Reverse DNS
  PTR: {
    example: '34.216.184.93.in-addr.arpa. 300 IN PTR example.com.';
    use: 'IP to domain mapping (reverse lookup)';
  };
}

// Common TXT record patterns
const txtRecordPatterns = {
  // SPF - Sender Policy Framework
  spf: 'v=spf1 include:_spf.google.com include:sendgrid.net -all',
  
  // DKIM selector
  dkim: 'v=DKIM1; k=rsa; p=MIGfMA0GCS...',
  
  // DMARC
  dmarc: 'v=DMARC1; p=reject; rua=mailto:dmarc@example.com',
  
  // Domain verification
  googleVerification: 'google-site-verification=abc123...',
  
  // Let's Encrypt challenge
  acmeChallenge: '_acme-challenge.example.com TXT "challenge-token"',
};
```

### DNS-Based Load Balancing

```typescript
// dns/load-balancing.ts

/**
 * DNS Load Balancing Strategies
 */

// 1. Round-Robin DNS - Multiple A records
const roundRobinConfig = {
  records: [
    { name: 'api.example.com', type: 'A', value: '10.0.1.1', ttl: 60 },
    { name: 'api.example.com', type: 'A', value: '10.0.1.2', ttl: 60 },
    { name: 'api.example.com', type: 'A', value: '10.0.1.3', ttl: 60 },
  ],
  // DNS resolvers will rotate through these
  // Simple but no health checking
};

// 2. Weighted Round-Robin (Route 53, Cloudflare)
const weightedConfig = {
  records: [
    { name: 'api.example.com', value: '10.0.1.1', weight: 70 }, // 70% traffic
    { name: 'api.example.com', value: '10.0.1.2', weight: 20 }, // 20% traffic
    { name: 'api.example.com', value: '10.0.1.3', weight: 10 }, // 10% traffic
  ],
  // Useful for gradual migration or capacity differences
};

// 3. GeoDNS - Location-based routing
const geoDnsConfig = {
  records: [
    { name: 'api.example.com', value: '10.0.1.1', region: 'us-east-1' },
    { name: 'api.example.com', value: '10.0.2.1', region: 'eu-west-1' },
    { name: 'api.example.com', value: '10.0.3.1', region: 'ap-southeast-1' },
  ],
  // Route users to nearest region
};

// 4. Latency-based routing
const latencyConfig = {
  records: [
    { name: 'api.example.com', value: '10.0.1.1', region: 'us-east-1' },
    { name: 'api.example.com', value: '10.0.2.1', region: 'eu-west-1' },
  ],
  // DNS provider measures latency and routes to fastest
};

// 5. Failover DNS
const failoverConfig = {
  primary: { name: 'api.example.com', value: '10.0.1.1', healthCheck: true },
  secondary: { name: 'api.example.com', value: '10.0.2.1', healthCheck: true },
  // Automatic failover when primary health check fails
};

// Implementing client-side DNS caching with health checks
class DnsLoadBalancer {
  private servers: string[];
  private currentIndex = 0;
  private healthyServers: Set<string>;
  
  constructor(servers: string[]) {
    this.servers = servers;
    this.healthyServers = new Set(servers);
    this.startHealthChecks();
  }
  
  getServer(): string {
    const healthy = [...this.healthyServers];
    if (healthy.length === 0) {
      // All unhealthy, try anyway
      return this.servers[this.currentIndex++ % this.servers.length];
    }
    
    // Round-robin through healthy servers
    return healthy[this.currentIndex++ % healthy.length];
  }
  
  private async startHealthChecks(): Promise<void> {
    setInterval(async () => {
      for (const server of this.servers) {
        const healthy = await this.checkHealth(server);
        if (healthy) {
          this.healthyServers.add(server);
        } else {
          this.healthyServers.delete(server);
        }
      }
    }, 10000); // Check every 10 seconds
  }
  
  private async checkHealth(server: string): Promise<boolean> {
    try {
      const response = await fetch(`http://${server}/health`, {
        signal: AbortSignal.timeout(5000),
      });
      return response.ok;
    } catch {
      return false;
    }
  }
}
```

### DNS Configuration Examples

```yaml
# AWS Route 53 Terraform configuration
# dns/route53.tf

resource "aws_route53_zone" "main" {
  name = "example.com"
}

# Simple A record
resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "www.example.com"
  type    = "A"
  ttl     = 300
  records = ["10.0.1.1"]
}

# Weighted routing
resource "aws_route53_record" "api_primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"
  ttl     = 60
  
  weighted_routing_policy {
    weight = 70
  }
  
  set_identifier = "primary"
  records        = ["10.0.1.1"]
}

resource "aws_route53_record" "api_secondary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"
  ttl     = 60
  
  weighted_routing_policy {
    weight = 30
  }
  
  set_identifier = "secondary"
  records        = ["10.0.1.2"]
}

# Failover with health check
resource "aws_route53_health_check" "primary" {
  fqdn              = "primary.example.com"
  port              = 443
  type              = "HTTPS"
  resource_path     = "/health"
  failure_threshold = 3
  request_interval  = 30
}

resource "aws_route53_record" "failover_primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"
  
  failover_routing_policy {
    type = "PRIMARY"
  }
  
  set_identifier  = "primary"
  health_check_id = aws_route53_health_check.primary.id
  records         = ["10.0.1.1"]
  ttl             = 60
}

resource "aws_route53_record" "failover_secondary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"
  
  failover_routing_policy {
    type = "SECONDARY"
  }
  
  set_identifier = "secondary"
  records        = ["10.0.2.1"]
  ttl            = 60
}

# Geolocation routing
resource "aws_route53_record" "geo_us" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "cdn.example.com"
  type    = "A"
  
  geolocation_routing_policy {
    country = "US"
  }
  
  set_identifier = "us"
  records        = ["10.0.1.1"]
  ttl            = 300
}

resource "aws_route53_record" "geo_eu" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "cdn.example.com"
  type    = "A"
  
  geolocation_routing_policy {
    continent = "EU"
  }
  
  set_identifier = "eu"
  records        = ["10.0.2.1"]
  ttl            = 300
}
```

---

## Real-World Scenarios

### Scenario 1: Zero-Downtime DNS Migration

```typescript
// dns/migration.ts

/**
 * Steps for zero-downtime DNS migration:
 * 
 * 1. Lower TTL days before migration (300s or less)
 * 2. Wait for old TTL to expire (propagation)
 * 3. Make the DNS change
 * 4. Verify propagation across resolvers
 * 5. Keep old server running for stragglers
 * 6. Raise TTL back to normal after confirmed
 */

interface MigrationPlan {
  domain: string;
  oldIp: string;
  newIp: string;
  currentTtl: number;
  migrationTtl: number;
}

async function executeMigration(plan: MigrationPlan): Promise<void> {
  console.log('=== DNS Migration Plan ===');
  console.log(`Domain: ${plan.domain}`);
  console.log(`Old IP: ${plan.oldIp} → New IP: ${plan.newIp}`);
  
  // Step 1: Lower TTL
  console.log('\n1. Lower TTL to', plan.migrationTtl, 'seconds');
  // await updateDnsRecord(plan.domain, { ttl: plan.migrationTtl });
  
  // Step 2: Wait for propagation
  const waitTime = plan.currentTtl;
  console.log(`\n2. Waiting ${waitTime} seconds for TTL expiration...`);
  await sleep(waitTime * 1000);
  
  // Step 3: Verify old TTL expired
  console.log('\n3. Checking propagation of new TTL...');
  let propagated = await checkPropagation(plan.domain, plan.oldIp);
  console.log('Propagation status:', propagated);
  
  // Step 4: Make the change
  console.log('\n4. Updating A record to new IP...');
  // await updateDnsRecord(plan.domain, { value: plan.newIp });
  
  // Step 5: Verify new IP propagation
  console.log('\n5. Waiting for new IP propagation...');
  await sleep(plan.migrationTtl * 2 * 1000);
  
  propagated = await checkPropagation(plan.domain, plan.newIp);
  const allPropagated = propagated.every(p => p.matches);
  
  if (allPropagated) {
    console.log('\n✅ Migration successful! All resolvers returning new IP.');
    
    // Step 6: Raise TTL back
    console.log('\n6. Raising TTL back to', plan.currentTtl);
    // await updateDnsRecord(plan.domain, { ttl: plan.currentTtl });
  } else {
    console.log('\n⚠️ Some resolvers still returning old IP:', 
      propagated.filter(p => !p.matches));
  }
}

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Scenario 2: Service Discovery with SRV Records

```typescript
// dns/service-discovery.ts
import * as dns from 'dns';
import { promisify } from 'util';

const resolveSrv = promisify(dns.resolveSrv);

interface ServiceInstance {
  host: string;
  port: number;
  priority: number;
  weight: number;
}

// Discover services via DNS SRV records
async function discoverService(serviceName: string): Promise<ServiceInstance[]> {
  // SRV record format: _service._protocol.domain
  // Example: _http._tcp.api.example.com
  
  try {
    const records = await resolveSrv(serviceName);
    
    // Sort by priority (lower = higher priority)
    // Within same priority, consider weight for load distribution
    return records
      .sort((a, b) => a.priority - b.priority)
      .map(r => ({
        host: r.name,
        port: r.port,
        priority: r.priority,
        weight: r.weight,
      }));
  } catch (error) {
    console.error('Service discovery failed:', error);
    return [];
  }
}

// Select instance based on priority and weight
function selectInstance(instances: ServiceInstance[]): ServiceInstance | null {
  if (instances.length === 0) return null;
  
  // Group by priority
  const lowestPriority = instances[0].priority;
  const samePriority = instances.filter(i => i.priority === lowestPriority);
  
  // Weighted random selection within same priority
  const totalWeight = samePriority.reduce((sum, i) => sum + i.weight, 0);
  let random = Math.random() * totalWeight;
  
  for (const instance of samePriority) {
    random -= instance.weight;
    if (random <= 0) {
      return instance;
    }
  }
  
  return samePriority[0];
}

// Example: Kubernetes-style service discovery
async function kubernetesServiceDiscovery(
  serviceName: string,
  namespace: string = 'default'
): Promise<ServiceInstance[]> {
  // Kubernetes creates SRV records like:
  // _port-name._protocol.service.namespace.svc.cluster.local
  
  const fqdn = `_http._tcp.${serviceName}.${namespace}.svc.cluster.local`;
  return discoverService(fqdn);
}
```

---

## Common Pitfalls

### 1. Forgetting to Lower TTL Before Changes

```typescript
// ❌ BAD: Changing IP with high TTL
// Current TTL: 86400 (24 hours)
// Users may see old IP for up to 24 hours!
await updateRecord('example.com', { value: newIp });

// ✅ GOOD: Lower TTL first, wait, then change
await updateRecord('example.com', { ttl: 300 }); // 5 minutes
await sleep(86400 * 1000); // Wait for old TTL to expire
await updateRecord('example.com', { value: newIp });
// Then raise TTL back after confirmed
```

### 2. CNAME at Zone Apex

```typescript
// ❌ BAD: CNAME at root domain (not allowed by RFC)
// example.com CNAME other.com  // INVALID!

// ✅ GOOD: Use ALIAS/ANAME (provider-specific) or A record
// example.com ALIAS lb.example.com (Route 53)
// or
// example.com A 10.0.1.1
```

### 3. Not Setting Up Health Checks with Load Balancing

```typescript
// ❌ BAD: Round-robin without health checks
const records = [
  { name: 'api.example.com', value: '10.0.1.1' }, // Could be down!
  { name: 'api.example.com', value: '10.0.1.2' },
];

// ✅ GOOD: Use DNS provider health checks
const records = [
  { name: 'api.example.com', value: '10.0.1.1', healthCheck: '/health' },
  { name: 'api.example.com', value: '10.0.1.2', healthCheck: '/health' },
];
// Unhealthy servers automatically removed from rotation
```

---

## Interview Questions

### Q1: Explain the DNS resolution process step by step.

**A:** 1) Browser checks its cache, then OS cache. 2) Query goes to configured DNS resolver (ISP or public like 8.8.8.8). 3) Resolver checks its cache; if miss, queries root servers. 4) Root server directs to TLD server (.com, .org). 5) TLD server directs to authoritative nameserver. 6) Authoritative server returns the IP. 7) Result cached at each level based on TTL.

### Q2: What's the difference between A, CNAME, and ALIAS records?

**A:** **A record** maps domain directly to IP address. **CNAME** (Canonical Name) aliases one domain to another - the resolver follows the chain. CNAME can't exist at zone apex or with other records. **ALIAS** (provider-specific) acts like CNAME but resolves at the DNS server, returning an A record - allows CNAME-like behavior at apex.

### Q3: How would you implement DNS-based failover?

**A:** 1) Configure primary and secondary records in DNS provider with health checks. 2) Set low TTL for faster failover. 3) Health checks ping endpoints (HTTP, TCP). 4) When primary fails health check, DNS automatically returns secondary IP. 5) Keep both environments ready and synchronized.

### Q4: What is TTL and how do you choose the right value?

**A:** TTL (Time To Live) is how long DNS responses are cached. Low TTL (60-300s): faster propagation for changes, more DNS queries, good before migrations. High TTL (3600-86400s): fewer queries, more caching, harder to change quickly. Choose based on how often you change records vs. query volume. Lower before planned changes, raise after.

---

## Quick Reference Checklist

### DNS Setup
- [ ] Configure primary and secondary nameservers
- [ ] Set appropriate TTL values
- [ ] Add CAA records for certificate control
- [ ] Configure SPF, DKIM, DMARC for email

### Load Balancing
- [ ] Set up health checks
- [ ] Choose appropriate strategy (round-robin, geo, weighted)
- [ ] Use low TTL for failover scenarios
- [ ] Test failover procedures

### Changes & Migrations
- [ ] Lower TTL before changes
- [ ] Wait for propagation
- [ ] Verify across multiple resolvers
- [ ] Keep old infrastructure running temporarily
- [ ] Raise TTL after confirmation

### Security
- [ ] Enable DNSSEC if supported
- [ ] Use CAA records
- [ ] Monitor for DNS hijacking
- [ ] Audit DNS records regularly

---

*Last updated: February 2026*

