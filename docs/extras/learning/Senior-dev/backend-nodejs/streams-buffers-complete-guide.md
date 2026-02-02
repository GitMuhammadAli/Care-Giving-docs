# Streams & Buffers - Complete Guide

> **MUST REMEMBER**: Streams process data piece by piece without loading everything into memory, making them essential for handling large files, network data, and real-time processing. Buffers are fixed-size chunks of memory that hold binary data. Together, they enable memory-efficient I/O operations.

---

## How to Explain Like a Senior Developer

"Streams are like a conveyor belt for data - instead of waiting for the entire shipment to arrive before processing, you handle each item as it comes through. This is crucial when dealing with gigabyte files or thousands of concurrent connections - you can't load everything into memory. Buffers are the containers on that conveyor belt, holding chunks of binary data. The key concept is backpressure - knowing when to slow down the conveyor because the processing station can't keep up. Master streams and you can build memory-efficient applications that handle massive data at scale."

---

## Core Implementation

### Understanding Buffers

```typescript
// buffers-basics.ts

// Creating buffers
const buf1 = Buffer.alloc(10); // 10-byte zero-filled buffer
const buf2 = Buffer.allocUnsafe(10); // 10-byte uninitialized (faster, but contains old memory)
const buf3 = Buffer.from('Hello'); // Buffer from string
const buf4 = Buffer.from([72, 101, 108, 108, 111]); // Buffer from array
const buf5 = Buffer.from('48656c6c6f', 'hex'); // Buffer from hex string

// Buffer operations
console.log(buf3.toString()); // 'Hello'
console.log(buf3.toString('hex')); // '48656c6c6f'
console.log(buf3.toString('base64')); // 'SGVsbG8='
console.log(buf3.length); // 5 bytes
console.log(buf3[0]); // 72 (ASCII for 'H')

// Comparing buffers
const a = Buffer.from('ABC');
const b = Buffer.from('ABC');
const c = Buffer.from('ABD');
console.log(a.equals(b)); // true
console.log(Buffer.compare(a, c)); // -1 (a < c)

// Concatenating buffers
const combined = Buffer.concat([buf3, Buffer.from(' World')]);
console.log(combined.toString()); // 'Hello World'

// Slicing (creates a view, not a copy!)
const slice = buf3.slice(0, 2);
slice[0] = 74; // Modifies original buffer!
console.log(buf3.toString()); // 'Jello'

// Copying (safe)
const copy = Buffer.alloc(5);
buf3.copy(copy);
copy[0] = 72;
console.log(buf3.toString()); // Still 'Jello'

// Buffer pool (why allocUnsafe is faster)
// Node.js maintains a pre-allocated pool of memory
// allocUnsafe reuses memory from this pool without zeroing
// alloc zeros the memory, which takes time
```

### Stream Types Overview

```typescript
// stream-types.ts
import { 
  Readable, 
  Writable, 
  Duplex, 
  Transform,
  pipeline
} from 'stream';
import { promisify } from 'util';

const pipelineAsync = promisify(pipeline);

/**
 * Stream Types:
 * 
 * Readable  - Source of data (fs.createReadStream, http.IncomingMessage)
 * Writable  - Destination for data (fs.createWriteStream, http.ServerResponse)
 * Duplex    - Both readable and writable (net.Socket, crypto streams)
 * Transform - Duplex that modifies data passing through (zlib, crypto)
 */

// Custom Readable Stream
class CounterStream extends Readable {
  private current = 0;
  private readonly max: number;
  
  constructor(max: number) {
    super({ objectMode: false }); // Binary mode
    this.max = max;
  }
  
  _read(size: number): void {
    if (this.current >= this.max) {
      this.push(null); // Signal end of stream
      return;
    }
    
    const chunk = `${this.current++}\n`;
    this.push(chunk); // Push data to internal buffer
  }
}

// Custom Writable Stream
class LoggerStream extends Writable {
  private bytesWritten = 0;
  
  _write(
    chunk: Buffer,
    encoding: BufferEncoding,
    callback: (error?: Error | null) => void
  ): void {
    this.bytesWritten += chunk.length;
    console.log(`Received: ${chunk.toString().trim()}`);
    callback(); // Signal completion
  }
  
  _final(callback: (error?: Error | null) => void): void {
    console.log(`Total bytes: ${this.bytesWritten}`);
    callback();
  }
}

// Custom Transform Stream
class UppercaseTransform extends Transform {
  _transform(
    chunk: Buffer,
    encoding: BufferEncoding,
    callback: (error?: Error | null, data?: Buffer) => void
  ): void {
    const uppercased = chunk.toString().toUpperCase();
    callback(null, Buffer.from(uppercased));
  }
}

// Custom Duplex Stream (echo server example)
class EchoStream extends Duplex {
  private buffer: Buffer[] = [];
  
  _read(size: number): void {
    const chunk = this.buffer.shift();
    this.push(chunk || null);
  }
  
  _write(
    chunk: Buffer,
    encoding: BufferEncoding,
    callback: (error?: Error | null) => void
  ): void {
    this.buffer.push(chunk);
    callback();
  }
}
```

### Backpressure Management

```typescript
// backpressure.ts
import * as fs from 'fs';
import { Readable, Writable } from 'stream';

/**
 * Backpressure occurs when the writable stream can't keep up
 * with the readable stream's data production rate.
 * 
 * If ignored, data accumulates in memory → memory exhaustion
 */

// ❌ BAD: Ignoring backpressure
function badPipe(): void {
  const readable = fs.createReadStream('huge-file.txt');
  const writable = fs.createWriteStream('output.txt');
  
  readable.on('data', (chunk) => {
    // write() returns false if internal buffer is full
    // but we ignore it → memory grows unbounded
    writable.write(chunk);
  });
}

// ✅ GOOD: Respecting backpressure manually
function goodPipe(): void {
  const readable = fs.createReadStream('huge-file.txt');
  const writable = fs.createWriteStream('output.txt');
  
  readable.on('data', (chunk) => {
    const canContinue = writable.write(chunk);
    
    if (!canContinue) {
      // Pause reading until writable drains
      readable.pause();
      
      writable.once('drain', () => {
        readable.resume();
      });
    }
  });
  
  readable.on('end', () => {
    writable.end();
  });
}

// ✅ BEST: Use pipe() or pipeline() - handles backpressure automatically
async function bestPipe(): Promise<void> {
  const { pipeline } = await import('stream/promises');
  
  await pipeline(
    fs.createReadStream('huge-file.txt'),
    fs.createWriteStream('output.txt')
  );
  
  console.log('Pipeline complete');
}

// Custom backpressure-aware producer
class BackpressureAwareProducer extends Readable {
  private data: string[];
  private index = 0;
  
  constructor(data: string[]) {
    super({ highWaterMark: 1024 }); // Buffer size threshold
    this.data = data;
  }
  
  _read(size: number): void {
    // Push data until buffer is full or data exhausted
    while (this.index < this.data.length) {
      const item = this.data[this.index++];
      const canPushMore = this.push(item + '\n');
      
      if (!canPushMore) {
        // Buffer is full, stop pushing
        // _read will be called again when buffer drains
        return;
      }
    }
    
    // End of data
    this.push(null);
  }
}
```

### Object Mode Streams

```typescript
// object-mode.ts
import { Transform, Readable, Writable } from 'stream';

/**
 * Object mode streams handle JavaScript objects instead of buffers
 * Useful for processing structured data
 */

interface User {
  id: number;
  name: string;
  email: string;
}

// Object mode readable
class UserStream extends Readable {
  private users: User[];
  private index = 0;
  
  constructor(users: User[]) {
    super({ objectMode: true });
    this.users = users;
  }
  
  _read(): void {
    if (this.index >= this.users.length) {
      this.push(null);
      return;
    }
    this.push(this.users[this.index++]);
  }
}

// Object mode transform
class UserValidator extends Transform {
  constructor() {
    super({ objectMode: true });
  }
  
  _transform(
    user: User,
    encoding: BufferEncoding,
    callback: (error?: Error | null, data?: User) => void
  ): void {
    if (!user.email.includes('@')) {
      callback(new Error(`Invalid email for user ${user.id}`));
      return;
    }
    
    callback(null, {
      ...user,
      email: user.email.toLowerCase(),
    });
  }
}

// Object mode writable
class UserSaver extends Writable {
  private saved: User[] = [];
  
  constructor() {
    super({ objectMode: true });
  }
  
  _write(
    user: User,
    encoding: BufferEncoding,
    callback: (error?: Error | null) => void
  ): void {
    this.saved.push(user);
    console.log(`Saved user: ${user.name}`);
    callback();
  }
  
  getSaved(): User[] {
    return this.saved;
  }
}

// Usage
async function processUsers(): Promise<void> {
  const { pipeline } = await import('stream/promises');
  
  const users: User[] = [
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'BOB@EXAMPLE.COM' },
  ];
  
  const saver = new UserSaver();
  
  await pipeline(
    new UserStream(users),
    new UserValidator(),
    saver
  );
  
  console.log('Processed:', saver.getSaved());
}
```

### File Streaming

```typescript
// file-streaming.ts
import * as fs from 'fs';
import * as zlib from 'zlib';
import { pipeline } from 'stream/promises';
import { createHash } from 'crypto';

// Stream file with progress
function streamWithProgress(
  inputPath: string,
  outputPath: string
): Promise<void> {
  return new Promise((resolve, reject) => {
    const stat = fs.statSync(inputPath);
    const totalSize = stat.size;
    let processedSize = 0;
    
    const readable = fs.createReadStream(inputPath);
    const writable = fs.createWriteStream(outputPath);
    
    readable.on('data', (chunk: Buffer) => {
      processedSize += chunk.length;
      const percentage = ((processedSize / totalSize) * 100).toFixed(2);
      process.stdout.write(`\rProgress: ${percentage}%`);
    });
    
    readable.pipe(writable);
    
    writable.on('finish', () => {
      console.log('\nComplete!');
      resolve();
    });
    
    writable.on('error', reject);
    readable.on('error', reject);
  });
}

// Compress file
async function compressFile(
  inputPath: string,
  outputPath: string
): Promise<void> {
  await pipeline(
    fs.createReadStream(inputPath),
    zlib.createGzip({ level: 9 }),
    fs.createWriteStream(outputPath)
  );
}

// Decompress file
async function decompressFile(
  inputPath: string,
  outputPath: string
): Promise<void> {
  await pipeline(
    fs.createReadStream(inputPath),
    zlib.createGunzip(),
    fs.createWriteStream(outputPath)
  );
}

// Calculate hash while streaming
async function hashFile(filePath: string): Promise<string> {
  return new Promise((resolve, reject) => {
    const hash = createHash('sha256');
    const stream = fs.createReadStream(filePath);
    
    stream.on('data', (chunk) => hash.update(chunk));
    stream.on('end', () => resolve(hash.digest('hex')));
    stream.on('error', reject);
  });
}

// Process large file line by line
import * as readline from 'readline';

async function processLines(filePath: string): Promise<number> {
  const stream = fs.createReadStream(filePath);
  const rl = readline.createInterface({
    input: stream,
    crlfDelay: Infinity, // Handle both \r\n and \n
  });
  
  let lineCount = 0;
  
  for await (const line of rl) {
    lineCount++;
    // Process each line
    if (line.startsWith('ERROR')) {
      console.log(`Error on line ${lineCount}: ${line}`);
    }
  }
  
  return lineCount;
}
```

### HTTP Streaming

```typescript
// http-streaming.ts
import * as http from 'http';
import * as https from 'https';
import * as fs from 'fs';
import { pipeline } from 'stream/promises';

// Stream response to file
async function downloadFile(
  url: string,
  outputPath: string
): Promise<void> {
  const response = await new Promise<http.IncomingMessage>((resolve, reject) => {
    https.get(url, resolve).on('error', reject);
  });
  
  if (response.statusCode !== 200) {
    throw new Error(`HTTP ${response.statusCode}`);
  }
  
  await pipeline(
    response,
    fs.createWriteStream(outputPath)
  );
}

// Stream server responses
const server = http.createServer((req, res) => {
  // Stream a large file
  if (req.url === '/download') {
    const filePath = './large-file.zip';
    const stat = fs.statSync(filePath);
    
    res.writeHead(200, {
      'Content-Type': 'application/octet-stream',
      'Content-Length': stat.size,
      'Content-Disposition': 'attachment; filename="file.zip"',
    });
    
    fs.createReadStream(filePath).pipe(res);
    return;
  }
  
  // Stream JSON array
  if (req.url === '/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    
    res.write('[');
    
    let first = true;
    const userStream = getUserStream(); // Returns readable stream
    
    userStream.on('data', (user: object) => {
      if (!first) res.write(',');
      first = false;
      res.write(JSON.stringify(user));
    });
    
    userStream.on('end', () => {
      res.write(']');
      res.end();
    });
    
    return;
  }
  
  // Stream request body
  if (req.method === 'POST' && req.url === '/upload') {
    const chunks: Buffer[] = [];
    
    req.on('data', (chunk: Buffer) => chunks.push(chunk));
    req.on('end', () => {
      const body = Buffer.concat(chunks);
      console.log('Received:', body.length, 'bytes');
      res.end('OK');
    });
    
    return;
  }
  
  res.writeHead(404);
  res.end('Not Found');
});

function getUserStream(): import('stream').Readable {
  // Return a readable stream of users
  const { Readable } = require('stream');
  const users = [{ id: 1 }, { id: 2 }];
  let index = 0;
  
  return new Readable({
    objectMode: true,
    read() {
      this.push(index < users.length ? users[index++] : null);
    }
  });
}
```

### Advanced Transform Patterns

```typescript
// transform-patterns.ts
import { Transform, TransformCallback } from 'stream';

// Batch transform - collect items, emit in batches
class BatchTransform<T> extends Transform {
  private batch: T[] = [];
  private readonly batchSize: number;
  
  constructor(batchSize: number) {
    super({ objectMode: true });
    this.batchSize = batchSize;
  }
  
  _transform(
    item: T,
    encoding: BufferEncoding,
    callback: TransformCallback
  ): void {
    this.batch.push(item);
    
    if (this.batch.length >= this.batchSize) {
      this.push([...this.batch]);
      this.batch = [];
    }
    
    callback();
  }
  
  _flush(callback: TransformCallback): void {
    if (this.batch.length > 0) {
      this.push([...this.batch]);
    }
    callback();
  }
}

// Throttle transform - rate limit items
class ThrottleTransform extends Transform {
  private readonly interval: number;
  private lastEmit = 0;
  
  constructor(itemsPerSecond: number) {
    super({ objectMode: true });
    this.interval = 1000 / itemsPerSecond;
  }
  
  _transform(
    chunk: unknown,
    encoding: BufferEncoding,
    callback: TransformCallback
  ): void {
    const now = Date.now();
    const elapsed = now - this.lastEmit;
    const wait = Math.max(0, this.interval - elapsed);
    
    setTimeout(() => {
      this.lastEmit = Date.now();
      callback(null, chunk);
    }, wait);
  }
}

// Filter transform
class FilterTransform<T> extends Transform {
  private readonly predicate: (item: T) => boolean;
  
  constructor(predicate: (item: T) => boolean) {
    super({ objectMode: true });
    this.predicate = predicate;
  }
  
  _transform(
    item: T,
    encoding: BufferEncoding,
    callback: TransformCallback
  ): void {
    if (this.predicate(item)) {
      callback(null, item);
    } else {
      callback(); // Skip item
    }
  }
}

// Map transform
class MapTransform<T, U> extends Transform {
  private readonly mapper: (item: T) => U | Promise<U>;
  
  constructor(mapper: (item: T) => U | Promise<U>) {
    super({ objectMode: true });
    this.mapper = mapper;
  }
  
  async _transform(
    item: T,
    encoding: BufferEncoding,
    callback: TransformCallback
  ): Promise<void> {
    try {
      const result = await this.mapper(item);
      callback(null, result);
    } catch (error) {
      callback(error as Error);
    }
  }
}

// Parallel transform with concurrency control
class ParallelTransform<T, U> extends Transform {
  private readonly processor: (item: T) => Promise<U>;
  private readonly concurrency: number;
  private running = 0;
  private queue: Array<{ item: T; callback: TransformCallback }> = [];
  
  constructor(processor: (item: T) => Promise<U>, concurrency: number) {
    super({ objectMode: true });
    this.processor = processor;
    this.concurrency = concurrency;
  }
  
  _transform(
    item: T,
    encoding: BufferEncoding,
    callback: TransformCallback
  ): void {
    if (this.running < this.concurrency) {
      this.process(item, callback);
    } else {
      this.queue.push({ item, callback });
    }
  }
  
  private async process(item: T, callback: TransformCallback): Promise<void> {
    this.running++;
    
    try {
      const result = await this.processor(item);
      this.push(result);
      callback();
    } catch (error) {
      callback(error as Error);
    } finally {
      this.running--;
      
      if (this.queue.length > 0) {
        const next = this.queue.shift()!;
        this.process(next.item, next.callback);
      }
    }
  }
  
  _flush(callback: TransformCallback): void {
    const checkComplete = (): void => {
      if (this.running === 0 && this.queue.length === 0) {
        callback();
      } else {
        setTimeout(checkComplete, 10);
      }
    };
    checkComplete();
  }
}
```

---

## Real-World Scenarios

### Scenario 1: CSV Processing Pipeline

```typescript
// csv-processor.ts
import * as fs from 'fs';
import { Transform } from 'stream';
import { pipeline } from 'stream/promises';
import * as readline from 'readline';

interface CSVRow {
  [key: string]: string;
}

// Parse CSV lines to objects
class CSVParser extends Transform {
  private headers: string[] | null = null;
  
  constructor() {
    super({ objectMode: true });
  }
  
  _transform(
    line: string,
    encoding: BufferEncoding,
    callback: (error?: Error | null, data?: CSVRow) => void
  ): void {
    const values = this.parseCSVLine(line);
    
    if (!this.headers) {
      this.headers = values;
      callback();
      return;
    }
    
    const row: CSVRow = {};
    this.headers.forEach((header, i) => {
      row[header] = values[i] || '';
    });
    
    callback(null, row);
  }
  
  private parseCSVLine(line: string): string[] {
    const result: string[] = [];
    let current = '';
    let inQuotes = false;
    
    for (const char of line) {
      if (char === '"') {
        inQuotes = !inQuotes;
      } else if (char === ',' && !inQuotes) {
        result.push(current.trim());
        current = '';
      } else {
        current += char;
      }
    }
    
    result.push(current.trim());
    return result;
  }
}

// Line reader as async generator
async function* readLines(filePath: string): AsyncGenerator<string> {
  const stream = fs.createReadStream(filePath);
  const rl = readline.createInterface({ input: stream });
  
  for await (const line of rl) {
    yield line;
  }
}

// Full CSV processing pipeline
async function processCSV(inputPath: string): Promise<void> {
  const { Readable } = await import('stream');
  
  const lineGenerator = readLines(inputPath);
  const readable = Readable.from(lineGenerator);
  
  await pipeline(
    readable,
    new CSVParser(),
    new FilterTransform<CSVRow>((row) => row.status === 'active'),
    new MapTransform<CSVRow, CSVRow>((row) => ({
      ...row,
      processed_at: new Date().toISOString(),
    })),
    new BatchTransform<CSVRow>(100),
    new Transform({
      objectMode: true,
      transform(batch: CSVRow[], encoding, callback) {
        console.log(`Processing batch of ${batch.length} rows`);
        // Save to database
        callback();
      }
    })
  );
}
```

### Scenario 2: Video Upload with Chunking

```typescript
// video-upload.ts
import * as fs from 'fs';
import { Transform } from 'stream';
import { pipeline } from 'stream/promises';
import * as crypto from 'crypto';

interface ChunkInfo {
  index: number;
  hash: string;
  size: number;
  data: Buffer;
}

class ChunkingTransform extends Transform {
  private readonly chunkSize: number;
  private buffer = Buffer.alloc(0);
  private chunkIndex = 0;
  
  constructor(chunkSizeMB: number = 5) {
    super({ objectMode: true });
    this.chunkSize = chunkSizeMB * 1024 * 1024;
  }
  
  _transform(
    data: Buffer,
    encoding: BufferEncoding,
    callback: (error?: Error | null) => void
  ): void {
    this.buffer = Buffer.concat([this.buffer, data]);
    
    while (this.buffer.length >= this.chunkSize) {
      const chunk = this.buffer.slice(0, this.chunkSize);
      this.buffer = this.buffer.slice(this.chunkSize);
      
      const info: ChunkInfo = {
        index: this.chunkIndex++,
        hash: crypto.createHash('md5').update(chunk).digest('hex'),
        size: chunk.length,
        data: chunk,
      };
      
      this.push(info);
    }
    
    callback();
  }
  
  _flush(callback: (error?: Error | null) => void): void {
    if (this.buffer.length > 0) {
      const info: ChunkInfo = {
        index: this.chunkIndex++,
        hash: crypto.createHash('md5').update(this.buffer).digest('hex'),
        size: this.buffer.length,
        data: this.buffer,
      };
      this.push(info);
    }
    callback();
  }
}

async function uploadVideo(filePath: string, uploadUrl: string): Promise<void> {
  const uploadChunk = async (chunk: ChunkInfo): Promise<void> => {
    // Simulate upload
    console.log(`Uploading chunk ${chunk.index}: ${chunk.size} bytes, hash: ${chunk.hash}`);
    await new Promise(resolve => setTimeout(resolve, 100));
  };
  
  await pipeline(
    fs.createReadStream(filePath),
    new ChunkingTransform(5), // 5MB chunks
    new ParallelTransform<ChunkInfo, void>(uploadChunk, 3) // 3 concurrent uploads
  );
  
  console.log('Upload complete');
}
```

### Scenario 3: Real-time Log Processing

```typescript
// log-processor.ts
import * as fs from 'fs';
import { Transform, Writable } from 'stream';
import * as readline from 'readline';

interface LogEntry {
  timestamp: Date;
  level: 'INFO' | 'WARN' | 'ERROR';
  message: string;
  metadata?: Record<string, unknown>;
}

interface LogStats {
  total: number;
  byLevel: Record<string, number>;
  errors: LogEntry[];
}

class LogParser extends Transform {
  constructor() {
    super({ objectMode: true });
  }
  
  _transform(
    line: string,
    encoding: BufferEncoding,
    callback: (error?: Error | null, data?: LogEntry) => void
  ): void {
    try {
      // Parse: [2026-02-02T10:30:00Z] ERROR: Something went wrong {"userId": 123}
      const match = line.match(
        /\[([^\]]+)\]\s+(INFO|WARN|ERROR):\s+(.+?)(?:\s+({.+}))?$/
      );
      
      if (!match) {
        callback();
        return;
      }
      
      const entry: LogEntry = {
        timestamp: new Date(match[1]),
        level: match[2] as LogEntry['level'],
        message: match[3],
        metadata: match[4] ? JSON.parse(match[4]) : undefined,
      };
      
      callback(null, entry);
    } catch (error) {
      callback(); // Skip malformed lines
    }
  }
}

class LogAggregator extends Writable {
  private stats: LogStats = {
    total: 0,
    byLevel: {},
    errors: [],
  };
  
  constructor() {
    super({ objectMode: true });
  }
  
  _write(
    entry: LogEntry,
    encoding: BufferEncoding,
    callback: (error?: Error | null) => void
  ): void {
    this.stats.total++;
    this.stats.byLevel[entry.level] = (this.stats.byLevel[entry.level] || 0) + 1;
    
    if (entry.level === 'ERROR') {
      this.stats.errors.push(entry);
    }
    
    callback();
  }
  
  getStats(): LogStats {
    return this.stats;
  }
}

// Watch and process log file in real-time
function watchLogFile(logPath: string): void {
  let position = 0;
  
  fs.watchFile(logPath, { interval: 1000 }, (curr, prev) => {
    if (curr.size > position) {
      const stream = fs.createReadStream(logPath, {
        start: position,
        end: curr.size - 1,
      });
      
      position = curr.size;
      
      const rl = readline.createInterface({ input: stream });
      
      rl.on('line', (line) => {
        // Process new line
        console.log('New log:', line);
      });
    }
  });
}
```

---

## Common Pitfalls

### 1. Memory Leaks with Unpiped Streams

```typescript
// ❌ BAD: Stream not consumed, buffers data indefinitely
function leakyStream(): void {
  const readable = getSomeReadableStream();
  
  readable.on('data', (chunk) => {
    // If processing is slow, data accumulates
    slowProcessing(chunk);
  });
  // No error handling, no end handling
}

// ✅ GOOD: Proper stream consumption
async function properStream(): Promise<void> {
  const { pipeline } = await import('stream/promises');
  
  await pipeline(
    getSomeReadableStream(),
    processChunk,
    outputStream
  );
}

function getSomeReadableStream(): import('stream').Readable {
  const { Readable } = require('stream');
  return new Readable({ read() { this.push(null); } });
}

function slowProcessing(chunk: Buffer): void {}
const processChunk = new Transform({ transform(c, e, cb) { cb(null, c); } });
const outputStream = new Writable({ write(c, e, cb) { cb(); } });
```

### 2. Not Handling Errors in Pipelines

```typescript
// ❌ BAD: Errors not handled
readable.pipe(transform).pipe(writable);
// If any stream errors, others may leak

// ✅ GOOD: Use pipeline with error handling
import { pipeline } from 'stream/promises';

try {
  await pipeline(readable, transform, writable);
} catch (error) {
  console.error('Pipeline failed:', error);
  // All streams properly destroyed
}
```

### 3. Modifying Sliced Buffers

```typescript
// ❌ BAD: Slice creates a view, not a copy
const original = Buffer.from('Hello');
const slice = original.slice(0, 3);
slice[0] = 74; // Modifies original too!
console.log(original.toString()); // 'Jello'

// ✅ GOOD: Use copy for independent buffer
const original2 = Buffer.from('Hello');
const copy = Buffer.alloc(3);
original2.copy(copy, 0, 0, 3);
copy[0] = 74;
console.log(original2.toString()); // 'Hello'
```

### 4. Ignoring highWaterMark

```typescript
// ❌ BAD: Default highWaterMark may be too small/large
const stream = fs.createReadStream('file.txt');
// Default is 64KB, may cause many small reads

// ✅ GOOD: Tune highWaterMark for your use case
const optimizedStream = fs.createReadStream('file.txt', {
  highWaterMark: 1024 * 1024, // 1MB for large files
});

// For object mode, highWaterMark is number of objects
const objectStream = new Readable({
  objectMode: true,
  highWaterMark: 100, // Buffer 100 objects
  read() {}
});
```

---

## Interview Questions

### Q1: What is backpressure and how do you handle it?

**A:** Backpressure occurs when a writable stream can't process data as fast as a readable stream produces it. Without handling, data accumulates in memory. Handle it by:
1. Using `pipe()` or `pipeline()` which auto-manage backpressure
2. Checking `write()` return value - false means buffer is full
3. Pausing readable when writable can't keep up, resuming on 'drain' event
4. Setting appropriate `highWaterMark` values

### Q2: What's the difference between Buffer.alloc() and Buffer.allocUnsafe()?

**A:** `Buffer.alloc(size)` creates a zero-filled buffer, which is safe but slower. `Buffer.allocUnsafe(size)` creates a buffer from the pre-allocated memory pool without zeroing, which is faster but may contain old data. Use `allocUnsafe` only when you'll immediately overwrite all bytes; otherwise use `alloc` for security.

### Q3: How do Transform streams differ from Duplex streams?

**A:** Both are readable and writable, but Transform streams have a specific relationship between input and output - data written is transformed and becomes readable output. Duplex streams have independent read/write sides (like a TCP socket). Transform is a specialized Duplex where `_transform()` processes each chunk.

### Q4: When should you use object mode streams?

**A:** Use object mode when processing structured data (JSON objects, database records, parsed lines). Benefits:
- Natural handling of JavaScript objects
- `highWaterMark` counts objects, not bytes
- No need for serialization between transforms
Avoid for binary data or when memory is critical (objects have overhead).

### Q5: How do you handle errors in a stream pipeline?

**A:** Use `pipeline()` from `stream/promises` which:
1. Propagates errors through all streams
2. Automatically destroys all streams on error
3. Returns a promise that rejects on any error
Avoid `.pipe()` chains without error handling as errors don't propagate and streams may leak.

---

## Quick Reference Checklist

### Buffer Operations
- [ ] Use `alloc` for safe initialization, `allocUnsafe` + fill for performance
- [ ] Remember `slice` creates views, use `copy` for independent buffers
- [ ] Use `Buffer.concat()` for combining buffers

### Stream Best Practices
- [ ] Always use `pipeline()` over manual `.pipe()` chains
- [ ] Handle backpressure - check `write()` returns and listen for 'drain'
- [ ] Set appropriate `highWaterMark` for your data size
- [ ] Implement `_flush()` in Transform for cleanup
- [ ] Use object mode for structured data processing

### Error Handling
- [ ] Use async `pipeline()` with try/catch
- [ ] Implement error events on custom streams
- [ ] Ensure streams are destroyed on error

### Performance
- [ ] Process in chunks, don't load entire files
- [ ] Use streams for files > 100MB
- [ ] Tune highWaterMark based on data patterns
- [ ] Consider parallel transforms for I/O-bound operations

---

*Last updated: February 2026*

