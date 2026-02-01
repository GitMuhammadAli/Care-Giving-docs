# 🗜️ Compression - Complete Guide

> A comprehensive guide to HTTP compression - Gzip vs Brotli, static vs dynamic compression, server configuration, and reducing transfer size by 70-90%.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "HTTP compression reduces transfer size by 70-90% using algorithms like Gzip (universal) or Brotli (20% smaller than Gzip) - the server compresses, the browser decompresses, and everyone saves bandwidth."

### The Compression Mental Model
```
WITHOUT COMPRESSION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Server                              Browser                    │
│  ┌──────────────────┐                ┌─────────────────────┐   │
│  │ main.js (500KB)  │ ─────────────► │ Receives 500KB      │   │
│  │ styles.css       │                │ Takes 2.5 seconds   │   │
│  │ (200KB)          │                │ on 3G               │   │
│  └──────────────────┘                └─────────────────────┘   │
│                                                                  │
│  Total transfer: 700KB                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

WITH COMPRESSION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Server                              Browser                    │
│  ┌──────────────────┐                ┌─────────────────────┐   │
│  │ main.js.br       │                │ Receives 70KB       │   │
│  │ (60KB - Brotli)  │ ─────────────► │ Decompresses to     │   │
│  │ styles.css.br    │                │ 700KB               │   │
│  │ (10KB - Brotli)  │                │ Takes 0.35 seconds  │   │
│  └──────────────────┘                └─────────────────────┘   │
│                                                                  │
│  Total transfer: 70KB (90% reduction!)                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Algorithm | Compression Ratio | Speed | Browser Support |
|-----------|------------------|-------|-----------------|
| **Brotli** | ~80-90% | Slower compress, fast decompress | 97%+ |
| **Gzip** | ~70-85% | Fast compress and decompress | 100% |
| **None** | 0% | N/A | N/A |

| File Type | Typical Savings |
|-----------|-----------------|
| JavaScript | 70-85% |
| CSS | 80-90% |
| HTML | 70-80% |
| JSON | 80-90% |
| SVG | 50-70% |
| Already compressed (JPEG, PNG, WOFF2) | 0-5% |

### The "Wow" Statement
> "Our site was serving uncompressed assets - 2MB of JavaScript and CSS. I enabled Brotli compression at the CDN level for static assets (pre-compressed at build time) and Gzip for dynamic API responses. Transfer size dropped from 2MB to 180KB - 91% reduction. Load time on 3G went from 10 seconds to under 2 seconds. The key insight was using static pre-compression for assets (maximum compression, computed once) and dynamic compression only for API responses. I also ensured we're not trying to compress already-compressed formats like JPEG and WOFF2."

---

## 📚 Core Concepts

### Gzip vs Brotli

```
ALGORITHM COMPARISON:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  GZIP (1992)                                                    │
│  ───────────                                                    │
│  • Universal support (100% browsers)                           │
│  • Fast compression and decompression                          │
│  • Good for dynamic content (real-time compression)            │
│  • Compression levels 1-9 (higher = smaller but slower)        │
│                                                                  │
│  BROTLI (2015)                                                  │
│  ─────────────                                                  │
│  • 15-20% better compression than Gzip                         │
│  • Slower compression (especially at high levels)              │
│  • Fast decompression                                          │
│  • Great for static pre-compressed assets                      │
│  • Compression levels 0-11                                     │
│  • 97%+ browser support                                        │
│                                                                  │
│  RECOMMENDATION:                                                │
│  • Static assets: Brotli (pre-compress at build time)          │
│  • Dynamic responses: Gzip (faster real-time compression)      │
│  • Fallback: Always have Gzip for older browsers               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Static vs Dynamic Compression

```
COMPRESSION STRATEGIES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  STATIC COMPRESSION (Pre-compressed)                           │
│  ────────────────────────────────────                          │
│  Build time: Generate .br and .gz files                        │
│                                                                  │
│  main.js (500KB)                                               │
│     └── main.js.br (50KB)   ← Pre-compressed                   │
│     └── main.js.gz (65KB)   ← Fallback                         │
│                                                                  │
│  ✓ Maximum compression (slow compression is fine)              │
│  ✓ No CPU cost at request time                                 │
│  ✓ Best for static assets (JS, CSS, HTML)                      │
│                                                                  │
│  DYNAMIC COMPRESSION (On-the-fly)                              │
│  ─────────────────────────────────                              │
│  Request time: Compress response before sending                │
│                                                                  │
│  API Request → Generate JSON → Compress → Send                 │
│                                                                  │
│  ✓ Works for dynamic content                                   │
│  ✗ CPU cost on every request                                   │
│  ✓ Use lower compression levels (speed matters)                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Build-Time Compression

```javascript
// ════════════════════════════════════════════════════════════════
// WEBPACK COMPRESSION PLUGIN
// ════════════════════════════════════════════════════════════════

const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
    plugins: [
        // Gzip compression
        new CompressionPlugin({
            filename: '[path][base].gz',
            algorithm: 'gzip',
            test: /\.(js|css|html|svg|json)$/,
            threshold: 1024,  // Only compress files > 1KB
            minRatio: 0.8,    // Only if 20%+ savings
        }),
        
        // Brotli compression
        new CompressionPlugin({
            filename: '[path][base].br',
            algorithm: 'brotliCompress',
            test: /\.(js|css|html|svg|json)$/,
            threshold: 1024,
            minRatio: 0.8,
            compressionOptions: {
                level: 11,  // Maximum compression for static files
            },
        }),
    ],
};

// ════════════════════════════════════════════════════════════════
// VITE COMPRESSION
// ════════════════════════════════════════════════════════════════

import viteCompression from 'vite-plugin-compression';

export default defineConfig({
    plugins: [
        // Gzip
        viteCompression({
            algorithm: 'gzip',
            ext: '.gz',
        }),
        // Brotli
        viteCompression({
            algorithm: 'brotliCompress',
            ext: '.br',
        }),
    ],
});

// ════════════════════════════════════════════════════════════════
// NEXT.JS - Automatic compression
// ════════════════════════════════════════════════════════════════

// next.config.js
module.exports = {
    compress: true,  // Enable gzip compression (default: true)
    
    // For Brotli, configure at CDN/reverse proxy level
    // Or use custom server
};
```

### Server Configuration

```nginx
# ════════════════════════════════════════════════════════════════
# NGINX CONFIGURATION
# ════════════════════════════════════════════════════════════════

# Enable gzip for dynamic responses
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;  # Balance between compression and CPU
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/json
    application/xml
    image/svg+xml;

# Don't compress already compressed files
gzip_disable "msie6";
gzip_min_length 1024;

# Enable Brotli (requires ngx_brotli module)
brotli on;
brotli_comp_level 6;
brotli_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/json
    application/xml
    image/svg+xml;

# Serve pre-compressed static files
location /static/ {
    # Try to serve .br first, then .gz, then original
    gzip_static on;
    brotli_static on;
    
    # Or manually:
    # try_files $uri.br $uri.gz $uri =404;
}
```

```typescript
// ════════════════════════════════════════════════════════════════
// EXPRESS.JS COMPRESSION
// ════════════════════════════════════════════════════════════════

import compression from 'compression';
import express from 'express';

const app = express();

// Enable gzip compression for all responses
app.use(compression({
    level: 6,                    // Compression level (1-9)
    threshold: 1024,             // Only compress if > 1KB
    filter: (req, res) => {
        // Don't compress already compressed types
        const type = res.getHeader('Content-Type');
        if (type && /image|video|audio/.test(type)) {
            return false;
        }
        return compression.filter(req, res);
    },
}));

// Serve pre-compressed static files
import expressStaticGzip from 'express-static-gzip';

app.use('/static', expressStaticGzip('public', {
    enableBrotli: true,
    orderPreference: ['br', 'gz'],
    serveStatic: {
        maxAge: '1y',
        immutable: true,
    },
}));
```

### CDN Configuration

```javascript
// ════════════════════════════════════════════════════════════════
// CLOUDFLARE (Automatic)
// ════════════════════════════════════════════════════════════════

// Cloudflare automatically compresses with Brotli and Gzip
// Just upload your files, compression is handled

// For static assets, upload pre-compressed for best results
// Cloudflare will serve the best version based on Accept-Encoding

// ════════════════════════════════════════════════════════════════
// AWS CLOUDFRONT + S3
// ════════════════════════════════════════════════════════════════

// S3: Upload both original and compressed versions
// main.js
// main.js.br (with Content-Encoding: br)
// main.js.gz (with Content-Encoding: gzip)

// CloudFront: Configure cache behavior
/*
{
    "CacheBehaviors": [{
        "Compress": true,  // Enable automatic compression
        "CachedMethods": ["GET", "HEAD"],
        // Forward Accept-Encoding header to origin
        "ForwardedValues": {
            "Headers": {
                "Items": ["Accept-Encoding"]
            }
        }
    }]
}
*/

// Or use Lambda@Edge for custom compression logic
```

---

## What to Compress

```
COMPRESSION BY FILE TYPE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ✓ COMPRESS (text-based, high compression ratio):              │
│  • JavaScript (.js)     - 70-85% savings                       │
│  • CSS (.css)           - 80-90% savings                       │
│  • HTML (.html)         - 70-80% savings                       │
│  • JSON (.json)         - 80-90% savings                       │
│  • SVG (.svg)           - 50-70% savings                       │
│  • XML (.xml)           - 70-80% savings                       │
│  • Plain text (.txt)    - 60-80% savings                       │
│                                                                  │
│  ✗ DON'T COMPRESS (already compressed):                        │
│  • JPEG, PNG, GIF, WebP - Already compressed images            │
│  • WOFF2                - Already compressed fonts             │
│  • MP4, WebM            - Already compressed video             │
│  • ZIP, GZIP, BR        - Already compressed archives          │
│  • PDF                  - Usually already compressed           │
│                                                                  │
│  Compressing already-compressed files:                         │
│  • Wastes CPU                                                  │
│  • Might actually increase size!                               │
│  • Provides no benefit                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "When would you use Gzip vs Brotli?"**
> "Brotli for static assets - pre-compress at build time with max compression (level 11), 15-20% smaller than Gzip. Gzip for dynamic content - faster compression for real-time responses. Always have Gzip fallback since Brotli isn't supported by all browsers. CDN typically handles this automatically based on Accept-Encoding header."

**Q: "What's the difference between static and dynamic compression?"**
> "Static: Pre-compress files at build time, serve pre-compressed versions. No CPU cost at request time, allows maximum compression. Dynamic: Compress on every request in real-time. Has CPU cost, use lower compression levels. Use static for assets (JS, CSS), dynamic for API responses."

**Q: "What files should NOT be compressed?"**
> "Already-compressed formats: JPEG, PNG, WebP, WOFF2, MP4, ZIP. Compressing them wastes CPU and provides 0-5% savings at best. Sometimes it increases size! Only compress text-based formats: JS, CSS, HTML, JSON, SVG, XML."

---

## Quick Reference

```
COMPRESSION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ALGORITHMS:                                                    │
│  • Brotli: Best ratio, slower compress, 97% support            │
│  • Gzip: Universal, fast, slightly larger                      │
│                                                                  │
│  STRATEGY:                                                      │
│  • Static assets: Brotli pre-compressed (level 11)             │
│  • Dynamic/API: Gzip real-time (level 6)                       │
│  • Always have Gzip fallback                                   │
│                                                                  │
│  COMPRESS:                                                      │
│  ✓ JS, CSS, HTML, JSON, SVG, XML                               │
│                                                                  │
│  DON'T COMPRESS:                                                │
│  ✗ JPEG, PNG, WebP, WOFF2, MP4, ZIP                           │
│                                                                  │
│  TYPICAL SAVINGS:                                               │
│  • JavaScript: 70-85%                                          │
│  • CSS: 80-90%                                                 │
│  • HTML: 70-80%                                                │
│  • JSON: 80-90%                                                │
│                                                                  │
│  CONFIG:                                                        │
│  • Nginx: gzip_static on; brotli_static on;                   │
│  • Express: compression() middleware                           │
│  • Build: CompressionPlugin                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


