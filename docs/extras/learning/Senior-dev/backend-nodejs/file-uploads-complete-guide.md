# File Uploads - Complete Guide

> **MUST REMEMBER**: Handle file uploads with streaming to avoid memory issues. Never trust client-provided filenames or MIME types. Use presigned URLs for direct-to-S3 uploads to bypass your server. Implement chunked uploads for large files to support resume capability. Always validate file types on the server and scan for malware in production.

---

## How to Explain Like a Senior Developer

"File uploads seem simple but have many gotchas. First, don't buffer entire files in memory - stream them or you'll crash on large files. Second, don't trust anything from the client - validate MIME types by reading magic bytes, sanitize filenames, and scan for malware. For large files, use presigned URLs so clients upload directly to S3, keeping load off your servers. For resumable uploads, implement chunked uploading with progress tracking. The key insight is that file upload is actually two problems: getting the bytes somewhere, and tracking/validating what was uploaded."

---

## Core Implementation

### Basic File Upload with Multer

```typescript
// file-upload-basic.ts
import express from 'express';
import multer from 'multer';
import path from 'path';
import crypto from 'crypto';

const app = express();

// Memory storage (for small files or immediate processing)
const memoryStorage = multer.memoryStorage();

// Disk storage (for larger files)
const diskStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, '/tmp/uploads');
  },
  filename: (req, file, cb) => {
    // Generate unique filename
    const uniqueSuffix = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname);
    cb(null, `${uniqueSuffix}${ext}`);
  },
});

// File filter
const fileFilter = (
  req: express.Request,
  file: Express.Multer.File,
  cb: multer.FileFilterCallback
) => {
  // Allowed MIME types
  const allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/webp',
    'application/pdf',
  ];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error(`File type ${file.mimetype} not allowed`));
  }
};

// Configure multer
const upload = multer({
  storage: diskStorage,
  fileFilter,
  limits: {
    fileSize: 10 * 1024 * 1024, // 10MB
    files: 5, // Max 5 files
  },
});

// Single file upload
app.post('/upload/single', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  res.json({
    filename: req.file.filename,
    size: req.file.size,
    mimetype: req.file.mimetype,
  });
});

// Multiple files upload
app.post('/upload/multiple', upload.array('files', 5), (req, res) => {
  const files = req.files as Express.Multer.File[];
  
  res.json({
    uploaded: files.map(f => ({
      filename: f.filename,
      size: f.size,
    })),
  });
});

// Handle multer errors
app.use((err: Error, req: express.Request, res: express.Response, next: express.NextFunction) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({ error: 'File too large' });
    }
    if (err.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).json({ error: 'Too many files' });
    }
  }
  
  res.status(500).json({ error: err.message });
});
```

### File Type Validation by Magic Bytes

```typescript
// file-validation.ts
import { fileTypeFromBuffer, fileTypeFromStream } from 'file-type';
import fs from 'fs';
import { Readable } from 'stream';

interface ValidationResult {
  valid: boolean;
  detectedType?: string;
  error?: string;
}

const ALLOWED_TYPES = new Map([
  ['image/jpeg', ['jpg', 'jpeg']],
  ['image/png', ['png']],
  ['image/webp', ['webp']],
  ['image/gif', ['gif']],
  ['application/pdf', ['pdf']],
]);

// Validate file from buffer
export async function validateFileBuffer(
  buffer: Buffer,
  expectedMime?: string
): Promise<ValidationResult> {
  const result = await fileTypeFromBuffer(buffer);
  
  if (!result) {
    return { valid: false, error: 'Could not determine file type' };
  }
  
  if (!ALLOWED_TYPES.has(result.mime)) {
    return {
      valid: false,
      detectedType: result.mime,
      error: `File type ${result.mime} not allowed`,
    };
  }
  
  // Verify expected type matches detected type
  if (expectedMime && result.mime !== expectedMime) {
    return {
      valid: false,
      detectedType: result.mime,
      error: `Expected ${expectedMime} but detected ${result.mime}`,
    };
  }
  
  return { valid: true, detectedType: result.mime };
}

// Validate file from stream (memory efficient)
export async function validateFileStream(
  stream: Readable
): Promise<ValidationResult> {
  const result = await fileTypeFromStream(stream);
  
  if (!result) {
    return { valid: false, error: 'Could not determine file type' };
  }
  
  if (!ALLOWED_TYPES.has(result.mime)) {
    return {
      valid: false,
      detectedType: result.mime,
      error: `File type ${result.mime} not allowed`,
    };
  }
  
  return { valid: true, detectedType: result.mime };
}

// Validate file from path
export async function validateFilePath(
  filePath: string
): Promise<ValidationResult> {
  const stream = fs.createReadStream(filePath);
  return validateFileStream(stream);
}

// Sanitize filename
export function sanitizeFilename(filename: string): string {
  // Remove path components
  const basename = filename.split(/[\\/]/).pop() || '';
  
  // Remove special characters, keep alphanumeric, dots, dashes, underscores
  const sanitized = basename.replace(/[^a-zA-Z0-9._-]/g, '_');
  
  // Limit length
  const maxLength = 255;
  if (sanitized.length > maxLength) {
    const ext = sanitized.split('.').pop() || '';
    const name = sanitized.slice(0, maxLength - ext.length - 1);
    return `${name}.${ext}`;
  }
  
  return sanitized;
}
```

### AWS S3 Presigned URLs

```typescript
// s3-presigned.ts
import {
  S3Client,
  PutObjectCommand,
  GetObjectCommand,
  DeleteObjectCommand,
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import crypto from 'crypto';

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const BUCKET = process.env.S3_BUCKET!;

interface PresignedUploadResult {
  uploadUrl: string;
  key: string;
  expiresIn: number;
}

// Generate presigned URL for upload
export async function getPresignedUploadUrl(
  filename: string,
  contentType: string,
  maxSize: number = 10 * 1024 * 1024
): Promise<PresignedUploadResult> {
  const key = generateObjectKey(filename);
  const expiresIn = 3600; // 1 hour
  
  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
    // Set max content length
    Metadata: {
      'original-filename': filename,
    },
  });
  
  const uploadUrl = await getSignedUrl(s3Client, command, { expiresIn });
  
  return { uploadUrl, key, expiresIn };
}

// Generate presigned URL for download
export async function getPresignedDownloadUrl(
  key: string,
  expiresIn: number = 3600
): Promise<string> {
  const command = new GetObjectCommand({
    Bucket: BUCKET,
    Key: key,
  });
  
  return getSignedUrl(s3Client, command, { expiresIn });
}

// Delete object
export async function deleteObject(key: string): Promise<void> {
  const command = new DeleteObjectCommand({
    Bucket: BUCKET,
    Key: key,
  });
  
  await s3Client.send(command);
}

// Generate unique object key
function generateObjectKey(filename: string): string {
  const date = new Date();
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const uniqueId = crypto.randomBytes(16).toString('hex');
  const ext = filename.split('.').pop();
  
  return `uploads/${year}/${month}/${day}/${uniqueId}.${ext}`;
}

// Express routes
import express from 'express';
const router = express.Router();

// Get presigned URL for upload
router.post('/upload/presigned', async (req, res) => {
  const { filename, contentType } = req.body;
  
  // Validate content type
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  if (!allowedTypes.includes(contentType)) {
    return res.status(400).json({ error: 'Invalid content type' });
  }
  
  try {
    const result = await getPresignedUploadUrl(filename, contentType);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: 'Failed to generate upload URL' });
  }
});

// Confirm upload complete (client calls after S3 upload)
router.post('/upload/confirm', async (req, res) => {
  const { key } = req.body;
  
  // Verify file exists in S3
  // Store reference in database
  // Trigger any post-processing
  
  res.json({ success: true });
});
```

### Chunked/Resumable Uploads

```typescript
// chunked-upload.ts
import express from 'express';
import fs from 'fs/promises';
import path from 'path';
import crypto from 'crypto';

const router = express.Router();
const UPLOAD_DIR = '/tmp/chunked-uploads';
const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB chunks

interface UploadSession {
  id: string;
  filename: string;
  totalSize: number;
  totalChunks: number;
  uploadedChunks: Set<number>;
  createdAt: Date;
}

// In-memory sessions (use Redis in production)
const uploadSessions = new Map<string, UploadSession>();

// Initialize upload session
router.post('/chunked/init', async (req, res) => {
  const { filename, totalSize, contentType } = req.body;
  
  const sessionId = crypto.randomBytes(16).toString('hex');
  const totalChunks = Math.ceil(totalSize / CHUNK_SIZE);
  
  const session: UploadSession = {
    id: sessionId,
    filename,
    totalSize,
    totalChunks,
    uploadedChunks: new Set(),
    createdAt: new Date(),
  };
  
  uploadSessions.set(sessionId, session);
  
  // Create session directory
  await fs.mkdir(path.join(UPLOAD_DIR, sessionId), { recursive: true });
  
  res.json({
    sessionId,
    chunkSize: CHUNK_SIZE,
    totalChunks,
  });
});

// Upload a chunk
router.post('/chunked/upload/:sessionId/:chunkIndex', express.raw({ limit: '6mb' }), async (req, res) => {
  const { sessionId, chunkIndex } = req.params;
  const index = parseInt(chunkIndex);
  
  const session = uploadSessions.get(sessionId);
  if (!session) {
    return res.status(404).json({ error: 'Upload session not found' });
  }
  
  if (index < 0 || index >= session.totalChunks) {
    return res.status(400).json({ error: 'Invalid chunk index' });
  }
  
  // Save chunk
  const chunkPath = path.join(UPLOAD_DIR, sessionId, `chunk_${index}`);
  await fs.writeFile(chunkPath, req.body);
  
  session.uploadedChunks.add(index);
  
  const progress = (session.uploadedChunks.size / session.totalChunks) * 100;
  const isComplete = session.uploadedChunks.size === session.totalChunks;
  
  res.json({
    chunkIndex: index,
    uploaded: session.uploadedChunks.size,
    total: session.totalChunks,
    progress: progress.toFixed(1),
    complete: isComplete,
  });
});

// Get upload status (for resuming)
router.get('/chunked/status/:sessionId', async (req, res) => {
  const { sessionId } = req.params;
  
  const session = uploadSessions.get(sessionId);
  if (!session) {
    return res.status(404).json({ error: 'Upload session not found' });
  }
  
  res.json({
    totalChunks: session.totalChunks,
    uploadedChunks: Array.from(session.uploadedChunks),
    missingChunks: Array.from(
      { length: session.totalChunks },
      (_, i) => i
    ).filter(i => !session.uploadedChunks.has(i)),
    progress: ((session.uploadedChunks.size / session.totalChunks) * 100).toFixed(1),
  });
});

// Complete upload (merge chunks)
router.post('/chunked/complete/:sessionId', async (req, res) => {
  const { sessionId } = req.params;
  
  const session = uploadSessions.get(sessionId);
  if (!session) {
    return res.status(404).json({ error: 'Upload session not found' });
  }
  
  if (session.uploadedChunks.size !== session.totalChunks) {
    return res.status(400).json({
      error: 'Upload incomplete',
      missing: session.totalChunks - session.uploadedChunks.size,
    });
  }
  
  // Merge chunks
  const finalPath = path.join(UPLOAD_DIR, `${sessionId}_${session.filename}`);
  const writeStream = await fs.open(finalPath, 'w');
  
  try {
    for (let i = 0; i < session.totalChunks; i++) {
      const chunkPath = path.join(UPLOAD_DIR, sessionId, `chunk_${i}`);
      const chunk = await fs.readFile(chunkPath);
      await writeStream.write(chunk);
    }
  } finally {
    await writeStream.close();
  }
  
  // Cleanup chunks
  await fs.rm(path.join(UPLOAD_DIR, sessionId), { recursive: true });
  uploadSessions.delete(sessionId);
  
  res.json({
    success: true,
    path: finalPath,
    size: session.totalSize,
  });
});

// Cancel upload
router.delete('/chunked/:sessionId', async (req, res) => {
  const { sessionId } = req.params;
  
  if (uploadSessions.has(sessionId)) {
    uploadSessions.delete(sessionId);
    await fs.rm(path.join(UPLOAD_DIR, sessionId), { recursive: true, force: true });
  }
  
  res.json({ success: true });
});

export { router as chunkedUploadRouter };
```

### Streaming Upload to S3

```typescript
// stream-to-s3.ts
import { S3Client } from '@aws-sdk/client-s3';
import { Upload } from '@aws-sdk/lib-storage';
import { PassThrough } from 'stream';
import express from 'express';
import busboy from 'busboy';

const s3Client = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.S3_BUCKET!;

const app = express();

// Stream upload directly to S3 (memory efficient)
app.post('/upload/stream', (req, res) => {
  const bb = busboy({
    headers: req.headers,
    limits: {
      fileSize: 100 * 1024 * 1024, // 100MB
    },
  });
  
  const uploads: Promise<void>[] = [];
  
  bb.on('file', (fieldname, file, info) => {
    const { filename, mimeType } = info;
    const key = `uploads/${Date.now()}_${filename}`;
    
    // Create pass-through stream
    const passThrough = new PassThrough();
    
    // Pipe file to pass-through
    file.pipe(passThrough);
    
    // Upload to S3 using multipart upload
    const upload = new Upload({
      client: s3Client,
      params: {
        Bucket: BUCKET,
        Key: key,
        Body: passThrough,
        ContentType: mimeType,
      },
      queueSize: 4,
      partSize: 5 * 1024 * 1024, // 5MB parts
    });
    
    upload.on('httpUploadProgress', (progress) => {
      console.log(`Upload progress: ${progress.loaded}/${progress.total}`);
    });
    
    uploads.push(
      upload.done().then(() => {
        console.log(`Uploaded ${key}`);
      })
    );
  });
  
  bb.on('close', async () => {
    try {
      await Promise.all(uploads);
      res.json({ success: true });
    } catch (error) {
      res.status(500).json({ error: 'Upload failed' });
    }
  });
  
  bb.on('error', (error) => {
    res.status(500).json({ error: error.message });
  });
  
  req.pipe(bb);
});
```

---

## Real-World Scenarios

### Scenario 1: Image Upload with Processing

```typescript
// image-upload-pipeline.ts
import express from 'express';
import multer from 'multer';
import sharp from 'sharp';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { validateFileBuffer } from './file-validation';

const s3 = new S3Client({ region: process.env.AWS_REGION });
const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 10 * 1024 * 1024 },
});

const app = express();

interface ProcessedImage {
  key: string;
  width: number;
  height: number;
  size: number;
}

app.post('/upload/image', upload.single('image'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  // Validate file type by magic bytes
  const validation = await validateFileBuffer(req.file.buffer);
  if (!validation.valid) {
    return res.status(400).json({ error: validation.error });
  }
  
  try {
    // Process image variants
    const variants = await processImageVariants(
      req.file.buffer,
      req.file.originalname
    );
    
    res.json({
      success: true,
      images: variants,
    });
  } catch (error) {
    console.error('Image processing error:', error);
    res.status(500).json({ error: 'Image processing failed' });
  }
});

async function processImageVariants(
  buffer: Buffer,
  originalName: string
): Promise<ProcessedImage[]> {
  const variants = [
    { suffix: 'original', width: null, height: null },
    { suffix: 'large', width: 1200, height: 1200 },
    { suffix: 'medium', width: 600, height: 600 },
    { suffix: 'thumb', width: 200, height: 200 },
  ];
  
  const baseName = originalName.replace(/\.[^.]+$/, '');
  const timestamp = Date.now();
  const results: ProcessedImage[] = [];
  
  for (const variant of variants) {
    let processed: sharp.Sharp = sharp(buffer);
    
    // Resize if dimensions specified
    if (variant.width && variant.height) {
      processed = processed.resize(variant.width, variant.height, {
        fit: 'inside',
        withoutEnlargement: true,
      });
    }
    
    // Convert to WebP for web optimization
    const output = await processed
      .webp({ quality: 85 })
      .toBuffer({ resolveWithObject: true });
    
    const key = `images/${timestamp}_${baseName}_${variant.suffix}.webp`;
    
    // Upload to S3
    await s3.send(new PutObjectCommand({
      Bucket: process.env.S3_BUCKET!,
      Key: key,
      Body: output.data,
      ContentType: 'image/webp',
    }));
    
    results.push({
      key,
      width: output.info.width,
      height: output.info.height,
      size: output.info.size,
    });
  }
  
  return results;
}
```

### Scenario 2: Multi-Part Form with Files and Data

```typescript
// multipart-form.ts
import express from 'express';
import multer from 'multer';
import { z } from 'zod';

const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 5 * 1024 * 1024, files: 5 },
});

const app = express();

// Schema for form data
const createPostSchema = z.object({
  title: z.string().min(1).max(200),
  description: z.string().max(5000),
  category: z.enum(['news', 'blog', 'announcement']),
  tags: z.string().transform(s => s.split(',').map(t => t.trim())),
});

// Handle form with both text fields and files
app.post('/posts', upload.array('attachments', 5), async (req, res) => {
  // Validate text fields
  const result = createPostSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation failed',
      details: result.error.flatten(),
    });
  }
  
  const { title, description, category, tags } = result.data;
  const files = req.files as Express.Multer.File[];
  
  // Process attachments
  const attachments = await Promise.all(
    files.map(async (file) => {
      const key = await uploadToS3(file);
      return {
        key,
        originalName: file.originalname,
        size: file.size,
        mimeType: file.mimetype,
      };
    })
  );
  
  // Create post in database
  const post = await createPost({
    title,
    description,
    category,
    tags,
    attachments,
  });
  
  res.status(201).json(post);
});

async function uploadToS3(file: Express.Multer.File): Promise<string> {
  return `uploads/${Date.now()}_${file.originalname}`;
}

async function createPost(data: object): Promise<object> {
  return { id: '1', ...data };
}
```

---

## Common Pitfalls

### 1. Trusting Client MIME Type

```typescript
// ❌ BAD: Trust client-provided MIME type
app.post('/upload', upload.single('file'), (req, res) => {
  if (req.file?.mimetype === 'image/jpeg') {
    // Could be malicious file with fake MIME type!
  }
});

// ✅ GOOD: Validate by magic bytes
app.post('/upload', upload.single('file'), async (req, res) => {
  const validation = await validateFileBuffer(req.file!.buffer);
  if (validation.detectedType !== 'image/jpeg') {
    return res.status(400).json({ error: 'Invalid file type' });
  }
});
```

### 2. Buffering Large Files in Memory

```typescript
// ❌ BAD: Buffer entire file
app.post('/upload', express.raw({ limit: '1gb' }), (req, res) => {
  const buffer = req.body; // OOM for large files!
  fs.writeFileSync('upload.bin', buffer);
});

// ✅ GOOD: Stream to disk or S3
app.post('/upload', (req, res) => {
  const writeStream = fs.createWriteStream('upload.bin');
  req.pipe(writeStream);
  writeStream.on('finish', () => res.json({ success: true }));
});
```

### 3. Not Cleaning Up Failed Uploads

```typescript
// ❌ BAD: Orphaned files on error
app.post('/upload', upload.single('file'), async (req, res) => {
  // File already saved to disk
  await processFile(req.file!.path); // Throws error
  // File remains on disk!
});

// ✅ GOOD: Cleanup on error
app.post('/upload', upload.single('file'), async (req, res) => {
  try {
    await processFile(req.file!.path);
    res.json({ success: true });
  } catch (error) {
    // Cleanup
    await fs.unlink(req.file!.path).catch(() => {});
    throw error;
  }
});

async function processFile(path: string): Promise<void> {}
```

---

## Interview Questions

### Q1: Why use presigned URLs instead of uploading through your server?

**A:** Presigned URLs allow clients to upload directly to S3, removing your server as a bottleneck. Benefits: reduced server load, lower bandwidth costs, faster uploads (direct to S3), and simpler scaling. Your server only generates the URL and confirms completion.

### Q2: How do you implement resumable uploads?

**A:** Split the file into chunks (5-10MB each) on the client. Track uploaded chunks server-side. Allow client to query which chunks are missing and resume from there. On completion, merge chunks into final file. S3 multipart uploads handle this natively.

### Q3: How do you validate file types securely?

**A:** Don't trust the client-provided MIME type or extension. Read the file's magic bytes (first few bytes that identify file type) using libraries like `file-type`. Validate against an allowlist of permitted types. For images, additionally try to process them with image libraries to ensure they're valid.

### Q4: How do you handle file uploads in a distributed system?

**A:** Use object storage (S3) as the central store - all servers can access files. For presigned uploads, any server can generate URLs. For server-side processing, use job queues to handle uploads asynchronously. Use CDN for serving files to reduce load.

---

## Quick Reference Checklist

### Security
- [ ] Validate file types by magic bytes, not MIME type
- [ ] Sanitize filenames
- [ ] Set file size limits
- [ ] Scan for malware in production
- [ ] Store uploads outside web root

### Performance
- [ ] Stream large files instead of buffering
- [ ] Use presigned URLs for direct S3 uploads
- [ ] Implement chunked uploads for large files
- [ ] Process files asynchronously via job queue

### User Experience
- [ ] Show upload progress
- [ ] Support resume for large files
- [ ] Validate file type on client before upload
- [ ] Handle errors gracefully

### Cleanup
- [ ] Delete temp files on error
- [ ] Set S3 lifecycle policies
- [ ] Clean up abandoned upload sessions

---

*Last updated: February 2026*

