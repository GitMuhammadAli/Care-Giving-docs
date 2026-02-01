# 🖼️ Image Optimization - Complete Guide

> A comprehensive guide to image optimization - WebP, AVIF, responsive images, lazy loading, CDN delivery, blur placeholders, and reducing image payload by 70%+.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Image optimization means serving the right format (WebP/AVIF), size (responsive srcset), and quality for each device and connection - typically reducing image payload by 50-80% while maintaining visual quality."

### The Image Optimization Mental Model
```
UNOPTIMIZED IMAGES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Mobile user loads page:                                        │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  hero.jpg (5MB, 4000x3000px, JPEG)                      │    │
│  │  product1.png (2MB, 2000x2000px)                        │    │
│  │  product2.png (2MB, 2000x2000px)                        │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Total: 9MB download for 400px wide phone screen!              │
│  Load time: 15+ seconds on 3G                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

OPTIMIZED IMAGES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Same mobile user:                                              │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  hero.avif (150KB, 800x600px, quality 80)               │    │
│  │  product1.webp (80KB, 400x400px)                        │    │
│  │  product2.webp (80KB, 400x400px, lazy loaded)           │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Total: 310KB (96% smaller!)                                   │
│  Load time: 2 seconds                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Format | Compression | Browser Support |
|--------|-------------|-----------------|
| **AVIF** | Best (50% smaller than JPEG) | 85%+ browsers |
| **WebP** | Great (25-35% smaller than JPEG) | 97%+ browsers |
| **JPEG** | Good baseline | 100% browsers |
| **PNG** | Lossless, large | 100% browsers |

| Metric | Target | Context |
|--------|--------|---------|
| LCP image | **<100KB** | Above-the-fold hero |
| Total images | **<500KB** | Initial page load |
| Quality setting | **75-85** | Visually lossless |
| Max width | **2x display** | For retina (e.g., 800px display → 1600px image) |

### The "Wow" Statement
> "Our e-commerce site loaded 8MB of images on the homepage - hero banner, product grid, all as uncompressed PNGs. I implemented a complete image pipeline: Sharp for server-side conversion to AVIF/WebP with JPEG fallback, srcset for responsive sizes, blur-hash placeholders, and lazy loading via Intersection Observer. Image payload dropped from 8MB to 600KB - 92% reduction. LCP improved from 4.2s to 1.1s. The key insight was using the `<picture>` element with AVIF as first source - browsers that support it get 50% smaller files automatically."

---

## 📚 Core Concepts

### Modern Image Formats

```
FORMAT COMPARISON:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Same image at similar visual quality:                         │
│                                                                  │
│  Format    │ Size      │ Support    │ Best For                 │
│  ──────────┼───────────┼────────────┼──────────────────────    │
│  AVIF      │ 50KB      │ 85%        │ Photos, best compression│
│  WebP      │ 75KB      │ 97%        │ Photos, wide support    │
│  JPEG      │ 100KB     │ 100%       │ Fallback for photos     │
│  PNG       │ 300KB     │ 100%       │ Transparency, graphics  │
│  SVG       │ 5KB       │ 100%       │ Icons, logos, vectors   │
│                                                                  │
│  DECISION TREE:                                                 │
│                                                                  │
│  Is it a vector/icon? ─────► SVG                               │
│         │                                                       │
│         No                                                      │
│         ↓                                                       │
│  Needs transparency? ─────► WebP (or PNG fallback)             │
│         │                                                       │
│         No                                                      │
│         ↓                                                       │
│  Photo/complex image? ─────► AVIF → WebP → JPEG fallback       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Responsive Images

```html
<!-- ════════════════════════════════════════════════════════════════ -->
<!-- SRCSET: Different sizes for different viewports                  -->
<!-- ════════════════════════════════════════════════════════════════ -->

<!-- Browser picks best size based on viewport and DPR -->
<img
    src="image-800.jpg"
    srcset="
        image-400.jpg 400w,
        image-800.jpg 800w,
        image-1200.jpg 1200w,
        image-1600.jpg 1600w
    "
    sizes="
        (max-width: 400px) 100vw,
        (max-width: 800px) 50vw,
        800px
    "
    alt="Product image"
    loading="lazy"
/>

<!-- 
sizes breakdown:
- Viewport ≤400px: image is 100% of viewport width
- Viewport ≤800px: image is 50% of viewport width  
- Larger: image is 800px

Browser calculates: "I need a 400px image at 2x DPR = 800w"
-->

<!-- ════════════════════════════════════════════════════════════════ -->
<!-- PICTURE: Different formats with fallback                         -->
<!-- ════════════════════════════════════════════════════════════════ -->

<picture>
    <!-- AVIF: Best compression, modern browsers -->
    <source
        type="image/avif"
        srcset="
            image-400.avif 400w,
            image-800.avif 800w,
            image-1200.avif 1200w
        "
        sizes="(max-width: 800px) 100vw, 800px"
    />
    
    <!-- WebP: Good compression, wide support -->
    <source
        type="image/webp"
        srcset="
            image-400.webp 400w,
            image-800.webp 800w,
            image-1200.webp 1200w
        "
        sizes="(max-width: 800px) 100vw, 800px"
    />
    
    <!-- JPEG: Fallback for old browsers -->
    <img
        src="image-800.jpg"
        srcset="
            image-400.jpg 400w,
            image-800.jpg 800w,
            image-1200.jpg 1200w
        "
        sizes="(max-width: 800px) 100vw, 800px"
        alt="Product image"
        loading="lazy"
        decoding="async"
    />
</picture>

<!-- ════════════════════════════════════════════════════════════════ -->
<!-- ART DIRECTION: Different crops for different viewports           -->
<!-- ════════════════════════════════════════════════════════════════ -->

<picture>
    <!-- Mobile: Square crop, focus on subject -->
    <source
        media="(max-width: 600px)"
        srcset="hero-mobile.webp"
    />
    
    <!-- Desktop: Wide banner -->
    <source
        media="(min-width: 601px)"
        srcset="hero-desktop.webp"
    />
    
    <img src="hero-desktop.jpg" alt="Hero banner" />
</picture>
```

### Server-Side Optimization with Sharp

```typescript
// ════════════════════════════════════════════════════════════════
// IMAGE PROCESSING PIPELINE
// ════════════════════════════════════════════════════════════════

import sharp from 'sharp';
import path from 'path';

interface ImageVariant {
    width: number;
    format: 'avif' | 'webp' | 'jpeg';
    quality: number;
}

const variants: ImageVariant[] = [
    // AVIF variants
    { width: 400, format: 'avif', quality: 80 },
    { width: 800, format: 'avif', quality: 80 },
    { width: 1200, format: 'avif', quality: 80 },
    { width: 1600, format: 'avif', quality: 80 },
    
    // WebP variants
    { width: 400, format: 'webp', quality: 82 },
    { width: 800, format: 'webp', quality: 82 },
    { width: 1200, format: 'webp', quality: 82 },
    { width: 1600, format: 'webp', quality: 82 },
    
    // JPEG fallback
    { width: 400, format: 'jpeg', quality: 85 },
    { width: 800, format: 'jpeg', quality: 85 },
    { width: 1200, format: 'jpeg', quality: 85 },
    { width: 1600, format: 'jpeg', quality: 85 },
];

async function optimizeImage(inputPath: string, outputDir: string) {
    const filename = path.parse(inputPath).name;
    const results = [];
    
    for (const variant of variants) {
        const outputPath = path.join(
            outputDir,
            `${filename}-${variant.width}.${variant.format}`
        );
        
        let pipeline = sharp(inputPath)
            .resize(variant.width, null, {
                withoutEnlargement: true, // Don't upscale
                fit: 'inside'
            });
        
        // Apply format-specific optimization
        switch (variant.format) {
            case 'avif':
                pipeline = pipeline.avif({ quality: variant.quality });
                break;
            case 'webp':
                pipeline = pipeline.webp({ quality: variant.quality });
                break;
            case 'jpeg':
                pipeline = pipeline.jpeg({ 
                    quality: variant.quality,
                    progressive: true,
                    mozjpeg: true
                });
                break;
        }
        
        await pipeline.toFile(outputPath);
        
        const stats = await sharp(outputPath).metadata();
        results.push({
            path: outputPath,
            width: stats.width,
            height: stats.height,
            size: (await fs.stat(outputPath)).size
        });
    }
    
    return results;
}

// ════════════════════════════════════════════════════════════════
// BLUR PLACEHOLDER GENERATION
// ════════════════════════════════════════════════════════════════

async function generateBlurPlaceholder(inputPath: string): Promise<string> {
    const buffer = await sharp(inputPath)
        .resize(10, 10, { fit: 'inside' })
        .blur()
        .toBuffer();
    
    return `data:image/jpeg;base64,${buffer.toString('base64')}`;
}

// Usage
const blurDataURL = await generateBlurPlaceholder('hero.jpg');
// Returns tiny base64 image for placeholder
```

### Next.js Image Component

```tsx
// ════════════════════════════════════════════════════════════════
// NEXT.JS IMAGE: Automatic optimization
// ════════════════════════════════════════════════════════════════

import Image from 'next/image';

// Basic usage - automatic optimization
<Image
    src="/hero.jpg"
    alt="Hero image"
    width={1200}
    height={600}
    priority  // Preload LCP image
/>

// Responsive with fill
<div style={{ position: 'relative', width: '100%', height: '400px' }}>
    <Image
        src="/banner.jpg"
        alt="Banner"
        fill
        style={{ objectFit: 'cover' }}
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
    />
</div>

// With blur placeholder
<Image
    src="/product.jpg"
    alt="Product"
    width={400}
    height={400}
    placeholder="blur"
    blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZ..."
/>

// External images (configure in next.config.js)
<Image
    src="https://cdn.example.com/image.jpg"
    alt="External"
    width={800}
    height={600}
    unoptimized={false}
/>

// ════════════════════════════════════════════════════════════════
// NEXT.CONFIG.JS CONFIGURATION
// ════════════════════════════════════════════════════════════════

// next.config.js
module.exports = {
    images: {
        domains: ['cdn.example.com', 'images.unsplash.com'],
        formats: ['image/avif', 'image/webp'],
        deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
        imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
        minimumCacheTTL: 60 * 60 * 24 * 30, // 30 days
    },
};
```

### Lazy Loading Images

```tsx
// ════════════════════════════════════════════════════════════════
// NATIVE LAZY LOADING
// ════════════════════════════════════════════════════════════════

// Simple - browser handles everything
<img 
    src="image.jpg" 
    alt="Description" 
    loading="lazy"
    decoding="async"
    width="800"   // Always set dimensions!
    height="600"
/>

// ════════════════════════════════════════════════════════════════
// PROGRESSIVE ENHANCEMENT WITH BLUR
// ════════════════════════════════════════════════════════════════

function ProgressiveImage({ src, blurSrc, alt, ...props }) {
    const [isLoaded, setIsLoaded] = useState(false);
    
    return (
        <div className="image-container" style={{ position: 'relative' }}>
            {/* Blur placeholder */}
            <img
                src={blurSrc}
                alt=""
                aria-hidden="true"
                className={`blur-placeholder ${isLoaded ? 'hidden' : ''}`}
                style={{
                    position: 'absolute',
                    inset: 0,
                    filter: 'blur(20px)',
                    transform: 'scale(1.1)',
                    transition: 'opacity 0.3s'
                }}
            />
            
            {/* Full image */}
            <img
                src={src}
                alt={alt}
                loading="lazy"
                onLoad={() => setIsLoaded(true)}
                style={{ opacity: isLoaded ? 1 : 0 }}
                {...props}
            />
        </div>
    );
}
```

---

## Common Pitfalls

```
IMAGE OPTIMIZATION PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. NO DIMENSIONS SET                                           │
│     Problem: Layout shift (CLS) when image loads               │
│     Solution: Always set width/height or aspect-ratio          │
│                                                                  │
│  2. SERVING DESKTOP IMAGES TO MOBILE                            │
│     Problem: 4000px image on 400px phone                       │
│     Solution: srcset with appropriate sizes                    │
│                                                                  │
│  3. ONLY JPEG/PNG                                               │
│     Problem: Missing 30-50% compression savings                │
│     Solution: picture element with AVIF/WebP + fallback        │
│                                                                  │
│  4. LAZY LOADING LCP IMAGE                                      │
│     Problem: Hero image loads late, bad LCP                    │
│     Solution: priority load for above-the-fold                 │
│                                                                  │
│  5. WRONG QUALITY SETTING                                       │
│     Problem: Too high = large files, too low = artifacts       │
│     Solution: 75-85 for photos, test visually                  │
│                                                                  │
│  6. NO CDN                                                      │
│     Problem: Images served from origin, slow globally          │
│     Solution: Use CDN with edge caching                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How do you optimize images for web?"**
> "Multi-pronged approach: 1) Modern formats - AVIF first, WebP fallback, JPEG last using `<picture>`. 2) Responsive sizes - srcset with breakpoints, never serve larger than needed. 3) Compression - quality 75-85, use tools like Sharp or Squoosh. 4) Lazy loading - native loading='lazy' for below-fold. 5) CDN delivery for global performance. Typically achieve 50-80% size reduction."

**Q: "What's the difference between srcset and picture?"**
> "srcset lets browser choose the best SIZE of the same image based on viewport and DPR. picture lets you specify different SOURCES - different formats (AVIF/WebP/JPEG) or different crops for different breakpoints. Use picture when you need format fallbacks or art direction, srcset when just serving different sizes."

**Q: "How do you prevent layout shift with images?"**
> "Always set dimensions. Options: explicit width/height attributes, CSS aspect-ratio property, or container with padding-bottom hack. For Next.js Image, dimensions are required. Also use placeholder blur to give visual feedback while loading."

---

## Quick Reference

```
IMAGE OPTIMIZATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  FORMATS (in order of preference):                              │
│  1. AVIF - Best compression, 85% support                       │
│  2. WebP - Good compression, 97% support                       │
│  3. JPEG - Fallback for photos                                 │
│  4. PNG - Only for transparency                                │
│  5. SVG - Icons and vectors                                    │
│                                                                  │
│  RESPONSIVE IMAGES:                                             │
│  • srcset: Different sizes of same image                       │
│  • sizes: Tell browser how big image will display              │
│  • picture: Different formats or art direction                 │
│                                                                  │
│  LOADING:                                                       │
│  • LCP image: priority/eager, no lazy                         │
│  • Below fold: loading="lazy"                                  │
│  • All images: Set width/height to prevent CLS                 │
│                                                                  │
│  QUALITY TARGETS:                                               │
│  • AVIF: 70-80                                                 │
│  • WebP: 75-85                                                 │
│  • JPEG: 80-85                                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


