# Content Negotiation - Complete Guide

> **MUST REMEMBER**: Content negotiation lets clients and servers agree on response format. Use Accept header for content type (JSON, XML), Accept-Language for language, Accept-Encoding for compression (gzip, br). Server responds with Content-Type, Content-Language, Content-Encoding. Return 406 Not Acceptable if you can't satisfy the request. Use Vary header for caching.

---

## How to Explain Like a Senior Developer

"Content negotiation is how clients and servers agree on format. The client says 'I prefer JSON, but XML is OK' via Accept headers. Server picks the best match and tells the client what it chose via Content-Type. This applies to content type (JSON vs XML), language (en vs fr), encoding (gzip vs brotli), and character set. Quality values (q=0.9) indicate preference. The Vary header is crucial for caching - it tells CDNs 'this response varies based on Accept header, so cache separately'. Most APIs just return JSON, but proper content negotiation makes APIs flexible and client-friendly."

---

## Core Implementation

### Accept Header Parsing

```typescript
// negotiation/accept-parser.ts

interface MediaType {
  type: string;        // e.g., "application"
  subtype: string;     // e.g., "json"
  fullType: string;    // e.g., "application/json"
  quality: number;     // 0-1, default 1
  parameters: Record<string, string>;
}

function parseAcceptHeader(header: string): MediaType[] {
  if (!header) return [];
  
  const mediaTypes: MediaType[] = header
    .split(',')
    .map(part => part.trim())
    .filter(Boolean)
    .map(part => {
      const [typeAndSubtype, ...params] = part.split(';').map(s => s.trim());
      const [type, subtype] = typeAndSubtype.split('/');
      
      let quality = 1;
      const parameters: Record<string, string> = {};
      
      for (const param of params) {
        const [key, value] = param.split('=').map(s => s.trim());
        if (key === 'q') {
          quality = parseFloat(value);
        } else {
          parameters[key] = value;
        }
      }
      
      return {
        type: type || '*',
        subtype: subtype || '*',
        fullType: typeAndSubtype,
        quality,
        parameters,
      };
    });
  
  // Sort by quality (highest first), then by specificity
  return mediaTypes.sort((a, b) => {
    // Higher quality first
    if (b.quality !== a.quality) return b.quality - a.quality;
    
    // More specific types first
    const aSpecificity = (a.type === '*' ? 0 : 1) + (a.subtype === '*' ? 0 : 1);
    const bSpecificity = (b.type === '*' ? 0 : 1) + (b.subtype === '*' ? 0 : 1);
    return bSpecificity - aSpecificity;
  });
}

// Check if server type matches client preference
function mediaTypeMatches(
  serverType: string,
  clientType: MediaType
): boolean {
  const [serverMain, serverSub] = serverType.split('/');
  
  if (clientType.type === '*' && clientType.subtype === '*') return true;
  if (clientType.type === serverMain && clientType.subtype === '*') return true;
  if (clientType.type === serverMain && clientType.subtype === serverSub) return true;
  
  return false;
}

// Find best match between client preferences and server capabilities
function negotiateContentType(
  acceptHeader: string,
  serverSupports: string[]
): string | null {
  const clientPreferences = parseAcceptHeader(acceptHeader);
  
  // No preference = accept anything
  if (clientPreferences.length === 0) {
    return serverSupports[0] || null;
  }
  
  // Find first match
  for (const preference of clientPreferences) {
    for (const serverType of serverSupports) {
      if (mediaTypeMatches(serverType, preference)) {
        return serverType;
      }
    }
  }
  
  return null; // No match = 406 Not Acceptable
}

// Usage
const acceptHeader = 'application/json, application/xml;q=0.9, */*;q=0.1';
const supported = ['application/json', 'application/xml', 'text/html'];
const best = negotiateContentType(acceptHeader, supported);
console.log(best); // 'application/json'
```

### Express Content Negotiation Middleware

```typescript
// negotiation/express-middleware.ts
import { Request, Response, NextFunction } from 'express';

interface ContentNegotiationOptions {
  supportedTypes: string[];
  supportedLanguages: string[];
  supportedEncodings: string[];
  defaultType: string;
  defaultLanguage: string;
}

const defaultOptions: ContentNegotiationOptions = {
  supportedTypes: ['application/json'],
  supportedLanguages: ['en'],
  supportedEncodings: ['identity', 'gzip', 'br'],
  defaultType: 'application/json',
  defaultLanguage: 'en',
};

export function contentNegotiation(options: Partial<ContentNegotiationOptions> = {}) {
  const config = { ...defaultOptions, ...options };
  
  return (req: Request, res: Response, next: NextFunction): void => {
    // Content-Type negotiation
    const acceptType = req.accepts(config.supportedTypes);
    if (!acceptType) {
      res.status(406).json({
        error: 'Not Acceptable',
        message: `Supported types: ${config.supportedTypes.join(', ')}`,
      });
      return;
    }
    
    // Language negotiation
    const acceptLang = req.acceptsLanguages(config.supportedLanguages);
    
    // Encoding negotiation
    const acceptEnc = req.acceptsEncodings(config.supportedEncodings);
    
    // Store negotiated values
    (req as any).negotiated = {
      contentType: acceptType || config.defaultType,
      language: acceptLang || config.defaultLanguage,
      encoding: acceptEnc || 'identity',
    };
    
    // Set Vary header for caching
    res.vary('Accept');
    res.vary('Accept-Language');
    res.vary('Accept-Encoding');
    
    next();
  };
}

// Response formatter based on negotiated type
export function formatResponse(
  req: Request,
  res: Response,
  data: object
): void {
  const negotiated = (req as any).negotiated || { contentType: 'application/json' };
  
  // Set content type and language
  res.type(negotiated.contentType);
  if (negotiated.language) {
    res.set('Content-Language', negotiated.language);
  }
  
  switch (negotiated.contentType) {
    case 'application/json':
      res.json(data);
      break;
      
    case 'application/xml':
      res.send(jsonToXml(data));
      break;
      
    case 'text/html':
      res.send(jsonToHtml(data));
      break;
      
    case 'text/csv':
      res.send(jsonToCsv(data));
      break;
      
    default:
      res.json(data);
  }
}

function jsonToXml(data: object): string {
  // Simplified XML conversion
  const toXml = (obj: any, name: string = 'root'): string => {
    if (Array.isArray(obj)) {
      return obj.map(item => toXml(item, 'item')).join('');
    }
    if (typeof obj === 'object' && obj !== null) {
      const content = Object.entries(obj)
        .map(([key, value]) => toXml(value, key))
        .join('');
      return `<${name}>${content}</${name}>`;
    }
    return `<${name}>${obj}</${name}>`;
  };
  
  return `<?xml version="1.0" encoding="UTF-8"?>${toXml(data)}`;
}

function jsonToHtml(data: object): string {
  return `<!DOCTYPE html>
<html>
<head><title>API Response</title></head>
<body><pre>${JSON.stringify(data, null, 2)}</pre></body>
</html>`;
}

function jsonToCsv(data: object): string {
  if (!Array.isArray(data)) return '';
  if (data.length === 0) return '';
  
  const headers = Object.keys(data[0]);
  const rows = data.map(row => 
    headers.map(h => JSON.stringify((row as any)[h] ?? '')).join(',')
  );
  
  return [headers.join(','), ...rows].join('\n');
}
```

### Language Negotiation

```typescript
// negotiation/language.ts
import { Request, Response, NextFunction } from 'express';

interface Translations {
  [locale: string]: {
    [key: string]: string;
  };
}

const translations: Translations = {
  'en': {
    'welcome': 'Welcome',
    'error.notFound': 'Resource not found',
    'error.unauthorized': 'Unauthorized access',
  },
  'es': {
    'welcome': 'Bienvenido',
    'error.notFound': 'Recurso no encontrado',
    'error.unauthorized': 'Acceso no autorizado',
  },
  'fr': {
    'welcome': 'Bienvenue',
    'error.notFound': 'Ressource non trouvée',
    'error.unauthorized': 'Accès non autorisé',
  },
};

const supportedLanguages = Object.keys(translations);

// Parse Accept-Language header
function parseAcceptLanguage(header: string): Array<{ locale: string; quality: number }> {
  if (!header) return [];
  
  return header
    .split(',')
    .map(part => {
      const [locale, ...params] = part.trim().split(';');
      let quality = 1;
      
      for (const param of params) {
        const match = param.trim().match(/^q=(.+)$/);
        if (match) quality = parseFloat(match[1]);
      }
      
      return { locale: locale.trim(), quality };
    })
    .sort((a, b) => b.quality - a.quality);
}

// Find best matching language
function negotiateLanguage(acceptHeader: string): string {
  const preferences = parseAcceptLanguage(acceptHeader);
  
  for (const pref of preferences) {
    // Exact match
    if (supportedLanguages.includes(pref.locale)) {
      return pref.locale;
    }
    
    // Language without region (e.g., "en-US" -> "en")
    const lang = pref.locale.split('-')[0];
    if (supportedLanguages.includes(lang)) {
      return lang;
    }
  }
  
  return 'en'; // Default
}

// Translation function
function t(locale: string, key: string, params?: Record<string, string>): string {
  const translation = translations[locale]?.[key] || translations['en'][key] || key;
  
  if (!params) return translation;
  
  // Simple parameter substitution
  return translation.replace(/\{\{(\w+)\}\}/g, (_, param) => params[param] || '');
}

// Express middleware
export function languageMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const acceptLanguage = req.headers['accept-language'] || '';
  const locale = negotiateLanguage(acceptLanguage);
  
  (req as any).locale = locale;
  (req as any).t = (key: string, params?: Record<string, string>) => t(locale, key, params);
  
  res.set('Content-Language', locale);
  res.vary('Accept-Language');
  
  next();
}

// Usage in route
import express from 'express';
const app = express();

app.use(languageMiddleware);

app.get('/welcome', (req: Request, res: Response) => {
  const t = (req as any).t;
  
  res.json({
    message: t('welcome'),
    locale: (req as any).locale,
  });
});
```

### Compression Negotiation

```typescript
// negotiation/compression.ts
import { Request, Response, NextFunction } from 'express';
import * as zlib from 'zlib';

// Parse Accept-Encoding header
function parseAcceptEncoding(header: string): Map<string, number> {
  const encodings = new Map<string, number>();
  
  if (!header) {
    encodings.set('identity', 1);
    return encodings;
  }
  
  for (const part of header.split(',')) {
    const [encoding, ...params] = part.trim().split(';');
    let quality = 1;
    
    for (const param of params) {
      const match = param.trim().match(/^q=(.+)$/);
      if (match) quality = parseFloat(match[1]);
    }
    
    encodings.set(encoding.trim(), quality);
  }
  
  return encodings;
}

// Choose best encoding
function negotiateEncoding(
  acceptHeader: string,
  supported: string[] = ['br', 'gzip', 'deflate', 'identity']
): string {
  const preferences = parseAcceptEncoding(acceptHeader);
  
  // Sort supported by client preference
  const ranked = supported
    .map(enc => ({
      encoding: enc,
      quality: preferences.get(enc) ?? preferences.get('*') ?? 0,
    }))
    .filter(e => e.quality > 0)
    .sort((a, b) => b.quality - a.quality);
  
  return ranked[0]?.encoding || 'identity';
}

// Compression middleware
export function compressionMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const acceptEncoding = req.headers['accept-encoding'] as string || '';
  const encoding = negotiateEncoding(acceptEncoding);
  
  // Store original send
  const originalSend = res.send.bind(res);
  
  res.send = function(body: any): Response {
    // Only compress if body is large enough
    const threshold = 1024; // 1KB
    const bodyString = typeof body === 'string' ? body : JSON.stringify(body);
    
    if (bodyString.length < threshold || encoding === 'identity') {
      return originalSend(body);
    }
    
    let compressed: Buffer;
    
    try {
      switch (encoding) {
        case 'br':
          compressed = zlib.brotliCompressSync(Buffer.from(bodyString));
          break;
        case 'gzip':
          compressed = zlib.gzipSync(Buffer.from(bodyString));
          break;
        case 'deflate':
          compressed = zlib.deflateSync(Buffer.from(bodyString));
          break;
        default:
          return originalSend(body);
      }
      
      res.set('Content-Encoding', encoding);
      res.set('Vary', 'Accept-Encoding');
      res.set('Content-Length', String(compressed.length));
      
      res.end(compressed);
      return res;
    } catch {
      return originalSend(body);
    }
  };
  
  next();
}
```

---

## Real-World Scenarios

### Scenario 1: Multi-Format API

```typescript
// api/multi-format.ts
import express, { Request, Response } from 'express';

const app = express();

interface User {
  id: number;
  name: string;
  email: string;
}

const users: User[] = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
];

// Format responders
const responders = {
  'application/json': (res: Response, data: any) => {
    res.json(data);
  },
  
  'application/xml': (res: Response, data: any) => {
    const xml = Array.isArray(data)
      ? `<?xml version="1.0"?><users>${data.map(u => 
          `<user><id>${u.id}</id><name>${u.name}</name><email>${u.email}</email></user>`
        ).join('')}</users>`
      : `<?xml version="1.0"?><user><id>${data.id}</id><name>${data.name}</name><email>${data.email}</email></user>`;
    res.type('application/xml').send(xml);
  },
  
  'text/csv': (res: Response, data: any) => {
    const items = Array.isArray(data) ? data : [data];
    const csv = 'id,name,email\n' + items.map(u => 
      `${u.id},"${u.name}","${u.email}"`
    ).join('\n');
    res.type('text/csv').send(csv);
  },
  
  'text/html': (res: Response, data: any) => {
    const items = Array.isArray(data) ? data : [data];
    const html = `<!DOCTYPE html>
<html>
<head><title>Users</title></head>
<body>
  <table border="1">
    <tr><th>ID</th><th>Name</th><th>Email</th></tr>
    ${items.map(u => `<tr><td>${u.id}</td><td>${u.name}</td><td>${u.email}</td></tr>`).join('')}
  </table>
</body>
</html>`;
    res.type('text/html').send(html);
  },
};

// Content negotiation endpoint
app.get('/users', (req: Request, res: Response) => {
  const supportedTypes = Object.keys(responders);
  const contentType = req.accepts(supportedTypes);
  
  if (!contentType) {
    res.status(406).json({
      error: 'Not Acceptable',
      supported: supportedTypes,
    });
    return;
  }
  
  res.vary('Accept');
  responders[contentType as keyof typeof responders](res, users);
});

app.get('/users/:id', (req: Request, res: Response) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    res.status(404).json({ error: 'User not found' });
    return;
  }
  
  const supportedTypes = Object.keys(responders);
  const contentType = req.accepts(supportedTypes);
  
  if (!contentType) {
    res.status(406).json({
      error: 'Not Acceptable',
      supported: supportedTypes,
    });
    return;
  }
  
  res.vary('Accept');
  responders[contentType as keyof typeof responders](res, user);
});

app.listen(3000);
```

### Scenario 2: CDN-Friendly Caching with Vary

```typescript
// caching/vary-header.ts
import express, { Request, Response } from 'express';

const app = express();

/**
 * Vary header tells caches to store separate versions
 * based on the listed request headers.
 */

app.get('/api/data', (req: Request, res: Response) => {
  // Set Vary for all negotiated headers
  res.vary('Accept');
  res.vary('Accept-Language');
  res.vary('Accept-Encoding');
  
  // Set cache headers
  res.set('Cache-Control', 'public, max-age=3600');
  
  // Content based on Accept
  const format = req.accepts(['json', 'xml']) || 'json';
  // Content based on Accept-Language
  const lang = req.acceptsLanguages(['en', 'es', 'fr']) || 'en';
  
  res.type(format === 'json' ? 'application/json' : 'application/xml');
  res.set('Content-Language', lang);
  
  // Send response
  // CDN will cache separate versions for:
  // - json/en, json/es, json/fr
  // - xml/en, xml/es, xml/fr
  // - With different encodings (gzip, br)
  
  res.json({ format, lang, data: 'example' });
});

// Avoid over-varying (cache fragmentation)
app.get('/api/public', (req: Request, res: Response) => {
  // Don't vary on User-Agent - creates too many cache versions
  // Don't vary on headers you don't actually use
  
  // Only vary on what you actually negotiate
  res.vary('Accept-Encoding'); // Compression only
  res.set('Cache-Control', 'public, max-age=86400');
  
  res.json({ data: 'public data' });
});

app.listen(3000);
```

---

## Common Pitfalls

### 1. Not Setting Vary Header

```typescript
// ❌ BAD: No Vary header - CDN caches one version for all
app.get('/api', (req, res) => {
  const format = req.accepts(['json', 'xml']);
  if (format === 'json') res.json(data);
  else res.type('xml').send(xmlData);
});

// ✅ GOOD: Vary header tells caches to store separate versions
app.get('/api', (req, res) => {
  res.vary('Accept'); // Critical for caching!
  const format = req.accepts(['json', 'xml']);
  if (format === 'json') res.json(data);
  else res.type('xml').send(xmlData);
});

const data = {};
const xmlData = '';
```

### 2. Returning 200 When Format Not Supported

```typescript
// ❌ BAD: Return JSON even if client only accepts XML
app.get('/api', (req, res) => {
  res.json(data); // Ignores Accept header!
});

// ✅ GOOD: Return 406 if can't satisfy request
app.get('/api', (req, res) => {
  const format = req.accepts(['json', 'xml']);
  if (!format) {
    res.status(406).json({ error: 'Not Acceptable' });
    return;
  }
  // Handle supported format...
});
```

### 3. Over-Varying Responses

```typescript
// ❌ BAD: Vary on everything - destroys cache efficiency
res.vary('Accept, Accept-Language, Accept-Encoding, User-Agent, Cookie');
// Creates millions of cache variations!

// ✅ GOOD: Only vary on headers you actually use for negotiation
res.vary('Accept'); // If you negotiate content type
res.vary('Accept-Language'); // If you negotiate language
res.vary('Accept-Encoding'); // Usually handled by CDN/proxy
```

---

## Interview Questions

### Q1: What is content negotiation and how does it work?

**A:** Content negotiation is the mechanism for serving different representations of a resource based on client preferences. Client sends Accept-* headers (Accept, Accept-Language, Accept-Encoding), server checks capabilities, picks best match, and indicates choice via Content-* headers. If no acceptable format exists, server returns 406 Not Acceptable.

### Q2: What is the Vary header and why is it important for caching?

**A:** Vary tells caches which request headers affect the response content. If you respond differently based on Accept-Language, set `Vary: Accept-Language`. Without it, a cache might serve a French response to an English user. Caches store separate copies for each combination of Vary header values. Over-varying fragments the cache.

### Q3: What's the difference between Accept and Content-Type headers?

**A:** **Accept** is a request header - client telling server what formats it can handle ("I want JSON"). **Content-Type** is typically a response header - server telling client what format it's sending ("This is JSON"). Content-Type can also be a request header when sending data (POST/PUT) to indicate the format of the request body.

### Q4: How do quality values (q=) work in Accept headers?

**A:** Quality values (0 to 1) indicate preference strength. `Accept: application/json, application/xml;q=0.9` means "prefer JSON (implied q=1), but XML is acceptable (q=0.9)". `q=0` means "don't send this format". Server should respect ordering and choose highest quality format it supports.

---

## Quick Reference Checklist

### Request Headers
- [ ] Accept - Content type preference
- [ ] Accept-Language - Language preference
- [ ] Accept-Encoding - Compression preference
- [ ] Accept-Charset - Character encoding preference

### Response Headers
- [ ] Content-Type - Actual content type
- [ ] Content-Language - Actual language
- [ ] Content-Encoding - Actual compression
- [ ] Vary - Headers that affect response

### Implementation
- [ ] Parse quality values correctly
- [ ] Return 406 if no acceptable format
- [ ] Always set Vary header
- [ ] Don't over-vary (cache fragmentation)

### Status Codes
- [ ] 200 OK - Successful negotiation
- [ ] 406 Not Acceptable - Can't satisfy Accept
- [ ] 415 Unsupported Media Type - Can't handle request Content-Type

---

*Last updated: February 2026*

