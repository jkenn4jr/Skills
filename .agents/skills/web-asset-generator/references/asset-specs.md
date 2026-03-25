# Web Asset Specifications

Platform-specific dimensions, formats, and constraints for web images and social assets.

---

## Open Graph (OG) / Meta Images

Used by Facebook, LinkedIn, Slack, iMessage, and most link-preview renderers.

| Spec | Value |
|------|-------|
| **Recommended size** | 1200 × 630 px |
| **Minimum size** | 600 × 315 px |
| **Aspect ratio** | 1.91:1 |
| **Max file size** | 8 MB (Facebook); keep under 1 MB for fast loading |
| **Format** | PNG or JPEG |
| **Safe zone** | Keep key content within center 1000 × 530 px |

**Meta tags:**
```html
<meta property="og:image" content="https://example.com/og/my-page.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:image:type" content="image/png" />
```

**Notes:**
- Facebook caches OG images aggressively — append `?v=2` or use a new filename when updating
- Text should be minimal; Facebook penalizes ad images with >20% text (no longer enforced but still best practice)
- Use Facebook Sharing Debugger to verify: https://developers.facebook.com/tools/debug/

---

## Twitter / X Cards

| Card Type | Dimensions | Aspect Ratio |
|-----------|-----------|--------------|
| **Summary** (small) | 120 × 120 px (thumbnail only) | 1:1 |
| **Summary Large Image** | 1200 × 628 px | ~1.91:1 |
| **Player card** | 1200 × 628 px | ~1.91:1 |
| **App card** | 800 × 418 px | ~1.91:1 |

**Recommended:** Summary Large Image at **1200 × 628 px** (or 1200 × 675 for in-stream feed display).

| Spec | Value |
|------|-------|
| **Format** | PNG, JPEG, GIF, WebP |
| **Max file size** | 5 MB |
| **Safe zone** | Keep text 60px from all edges (platform crops corners on mobile) |

**Meta tags:**
```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:image" content="https://example.com/og/my-page.png" />
<meta name="twitter:image:alt" content="Description of image" />
```

**Notes:**
- Twitter validator: https://cards-dev.twitter.com/validator
- GIFs animate on Twitter; use for attention-grabbing posts
- Images must be publicly accessible (no auth, no redirects to login)

---

## LinkedIn

### Link Post Preview Image

| Spec | Value |
|------|-------|
| **Recommended size** | 1200 × 628 px |
| **Aspect ratio** | 1.91:1 |
| **Format** | PNG or JPEG |
| **Max file size** | 5 MB |

LinkedIn reads OG image tags — a properly set OG image will appear in LinkedIn link previews.

### Company Page Banner

| Spec | Value |
|------|-------|
| **Size** | 1128 × 191 px |
| **Format** | PNG or JPEG |
| **Max file size** | 4 MB |

### Personal Profile Banner

| Spec | Value |
|------|-------|
| **Size** | 1584 × 396 px |
| **Format** | PNG or JPEG |

### Article Header Image

| Spec | Value |
|------|-------|
| **Recommended size** | 1200 × 644 px |
| **Minimum size** | 744 × 400 px |

---

## Favicons

Required sizes for cross-platform favicon support:

| File | Size | Purpose |
|------|------|---------|
| `favicon.ico` | 16×16, 32×32 (multi-size ICO) | Browsers, tab icon |
| `favicon-16x16.png` | 16 × 16 px | Explicit 16px |
| `favicon-32x32.png` | 32 × 32 px | Explicit 32px |
| `apple-touch-icon.png` | 180 × 180 px | iOS home screen |
| `android-chrome-192x192.png` | 192 × 192 px | Android home screen |
| `android-chrome-512x512.png` | 512 × 512 px | Android splash screen |
| `mstile-150x150.png` | 150 × 150 px | Windows tile (optional) |

**Source image recommendation:** Start with a 512×512 PNG with transparent background (RGBA mode).

**HTML:**
```html
<link rel="icon" type="image/x-icon" href="/favicon.ico" />
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
```

**Web manifest (for Android/PWA):**
```json
{
  "icons": [
    { "src": "/android-chrome-192x192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/android-chrome-512x512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

---

## YouTube Thumbnails

| Spec | Value |
|------|-------|
| **Recommended size** | 1280 × 720 px |
| **Minimum width** | 640 px |
| **Aspect ratio** | 16:9 |
| **Format** | PNG, JPEG, GIF, BMP, WebP |
| **Max file size** | 2 MB |
| **Safe zone** | Keep text/faces within center 1080 × 608 px |

**Notes:**
- YouTube shows thumbnails at many sizes — design for readability at 320px wide
- High contrast text with drop shadow performs best
- Include a face where possible (eyes toward camera) — higher CTR
- Avoid excessive text; YouTube's algorithm may deprioritize clickbait-style thumbnails

---

## App Store Icons

### iOS (App Store Connect)

| Spec | Value |
|------|-------|
| **Required size** | 1024 × 1024 px |
| **Format** | PNG |
| **Background** | Must not be transparent (iOS will reject transparent icons) |
| **Corners** | Do NOT round corners in the source — iOS applies mask automatically |

iOS also uses smaller sizes in various contexts (automatically scaled):
60×60 @1x, 60×60 @2x (120px), 60×60 @3x (180px), 76×76 @1x (iPad), etc.

### Google Play (Android)

| Spec | Value |
|------|-------|
| **Required size** | 512 × 512 px |
| **Format** | PNG, 32-bit |
| **Max file size** | 1 MB |
| **Background** | Can be transparent |

---

## Email Images

| Spec | Value |
|------|-------|
| **Hero image width** | 600 px (standard email width) |
| **Hero image height** | 200–300 px recommended |
| **Max file size** | 200 KB per image (total email under 1 MB) |
| **Format** | PNG or JPEG (avoid SVG — poor email client support) |
| **Resolution** | 72 DPI for screen; 2x (1200px) for retina email clients |

**Notes:**
- Many email clients block images by default — never put critical info in images only
- Always include `alt` text
- Avoid animated GIFs over 500KB — they are blocked by Outlook

---

## Social Media Post Images

### Instagram

| Placement | Dimensions | Aspect Ratio |
|-----------|-----------|--------------|
| Square post | 1080 × 1080 px | 1:1 |
| Portrait post | 1080 × 1350 px | 4:5 |
| Landscape post | 1080 × 566 px | 1.91:1 |
| Story / Reel | 1080 × 1920 px | 9:16 |

### Facebook

| Placement | Dimensions |
|-----------|-----------|
| Square post | 1080 × 1080 px |
| Landscape post | 1200 × 630 px |
| Story | 1080 × 1920 px |
| Event cover | 1200 × 628 px |

### Pinterest

| Spec | Value |
|------|-------|
| **Recommended** | 1000 × 1500 px (2:3 ratio) |
| **Square** | 1000 × 1000 px |
| **Long pin** | 1000 × 2100 px (max) |

---

## General Image Optimization Tips

### File size targets
- OG / social images: < 300 KB (ideal), < 1 MB (acceptable)
- Favicons: < 50 KB
- Email images: < 200 KB per image
- App icons: < 500 KB

### Format decision guide
| Use case | Format | Why |
|----------|--------|-----|
| Text on solid background | PNG | Lossless, sharp text |
| Photos / photographic backgrounds | JPEG (quality 85) | Good compression |
| Transparency needed | PNG | JPEG doesn't support alpha |
| Modern web delivery | WebP | 25-35% smaller than PNG/JPEG |
| Animated | GIF or WebP (animated) | Wide support |

### Compression tools
- `Pillow`: `img.save("out.jpg", quality=85, optimize=True)`
- `pngquant`: Lossy PNG compression (up to 70% smaller)
- `cwebp`: Convert to WebP from CLI
- `squoosh`: Web UI for batch compression
