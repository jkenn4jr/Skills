# Python Pillow Guide for Web Asset Generation

Practical patterns and ready-to-use code for generating web images with [Pillow](https://pillow.readthedocs.io/).

```bash
pip install Pillow
```

---

## Core Concepts

### Image modes

| Mode | Channels | Use when |
|------|----------|---------|
| `RGB` | Red, Green, Blue | JPEG output, no transparency |
| `RGBA` | Red, Green, Blue, Alpha | PNG output with transparency |
| `L` | Luminance | Grayscale |

Always match mode to output format:
```python
img = Image.new("RGBA", (1200, 630), (0, 0, 0, 0))  # transparent PNG
img = Image.new("RGB", (1200, 630), (30, 30, 30))    # dark JPEG/PNG
```

### Coordinate system

- Origin `(0, 0)` is **top-left**
- X increases to the right, Y increases downward
- `(width, height)` = bottom-right corner

---

## Template: OG Image Generator

Complete script for generating blog post OG images. Adapt configuration at the top.

```python
#!/usr/bin/env python3
"""
OG Image Generator
Generates 1200x630 Open Graph images for blog posts.

Usage:
  # Single image
  python generate_og.py --title "My Post Title" --category "Engineering" --output output/my-post.png

  # Batch from JSON
  python generate_og.py --input data/posts.json --output-dir output/og/
"""

import json
import argparse
from pathlib import Path
from PIL import Image, ImageDraw, ImageFont

# ── Configuration ─────────────────────────────────────────────────────────────
WIDTH, HEIGHT = 1200, 630
BG_COLOR = (18, 18, 18)          # Dark background
ACCENT_COLOR = (99, 102, 241)    # Indigo accent
TEXT_COLOR = (255, 255, 255)     # White
MUTED_COLOR = (160, 160, 160)    # Gray for secondary text

LOGO_PATH = "assets/logo.png"
FONT_BOLD = "fonts/Inter-Bold.ttf"
FONT_REGULAR = "fonts/Inter-Regular.ttf"

TITLE_SIZE = 64
META_SIZE = 28
PADDING = 80
# ──────────────────────────────────────────────────────────────────────────────


def load_font(path: str, size: int) -> ImageFont.FreeTypeFont:
    """Load a TTF font, falling back to default if file not found."""
    try:
        return ImageFont.truetype(path, size)
    except (IOError, OSError):
        print(f"Warning: Font not found at {path}, using default font")
        return ImageFont.load_default()


def wrap_text(draw: ImageDraw.Draw, text: str, font: ImageFont.FreeTypeFont, max_width: int) -> list[str]:
    """Word-wrap text to fit within max_width pixels."""
    words = text.split()
    lines = []
    current_line = ""

    for word in words:
        test_line = f"{current_line} {word}".strip()
        bbox = draw.textbbox((0, 0), test_line, font=font)
        if bbox[2] > max_width and current_line:
            lines.append(current_line)
            current_line = word
        else:
            current_line = test_line

    if current_line:
        lines.append(current_line)
    return lines


def draw_rounded_rect(draw: ImageDraw.Draw, xy: tuple, radius: int, fill: tuple) -> None:
    """Draw a filled rounded rectangle."""
    x1, y1, x2, y2 = xy
    draw.rectangle([x1 + radius, y1, x2 - radius, y2], fill=fill)
    draw.rectangle([x1, y1 + radius, x2, y2 - radius], fill=fill)
    draw.ellipse([x1, y1, x1 + radius * 2, y1 + radius * 2], fill=fill)
    draw.ellipse([x2 - radius * 2, y1, x2, y1 + radius * 2], fill=fill)
    draw.ellipse([x1, y2 - radius * 2, x1 + radius * 2, y2], fill=fill)
    draw.ellipse([x2 - radius * 2, y2 - radius * 2, x2, y2], fill=fill)


def generate_og_image(title: str, category: str = "", author: str = "", output_path: str = "output.png") -> None:
    """Generate a single OG image."""
    # Create base image
    img = Image.new("RGB", (WIDTH, HEIGHT), BG_COLOR)
    draw = ImageDraw.Draw(img)

    # Load fonts
    title_font = load_font(FONT_BOLD, TITLE_SIZE)
    meta_font = load_font(FONT_REGULAR, META_SIZE)
    category_font = load_font(FONT_BOLD, 22)

    # Draw subtle bottom gradient strip
    for y in range(HEIGHT - 200, HEIGHT):
        alpha = int(80 * (y - (HEIGHT - 200)) / 200)
        draw.line([(0, y), (WIDTH, y)], fill=(ACCENT_COLOR[0], ACCENT_COLOR[1], ACCENT_COLOR[2]))

    # Paste logo (top-left)
    logo_path = Path(LOGO_PATH)
    if logo_path.exists():
        logo = Image.open(logo_path).convert("RGBA")
        logo_h = 48
        logo_w = int(logo.width * (logo_h / logo.height))
        logo = logo.resize((logo_w, logo_h), Image.LANCZOS)
        img.paste(logo, (PADDING, PADDING), logo)

    # Category pill (if provided)
    y_offset = PADDING + 80
    if category:
        pill_text = category.upper()
        bbox = draw.textbbox((0, 0), pill_text, font=category_font)
        pill_w = bbox[2] - bbox[0] + 32
        pill_h = bbox[3] - bbox[1] + 16
        draw_rounded_rect(draw, (PADDING, y_offset, PADDING + pill_w, y_offset + pill_h), radius=8, fill=ACCENT_COLOR)
        draw.text((PADDING + 16, y_offset + 8), pill_text, font=category_font, fill=(255, 255, 255))
        y_offset += pill_h + 32

    # Title text (word-wrapped)
    max_text_width = WIDTH - (PADDING * 2)
    lines = wrap_text(draw, title, title_font, max_text_width)
    # Limit to 3 lines, truncate if longer
    if len(lines) > 3:
        lines = lines[:3]
        lines[-1] = lines[-1].rstrip() + "…"

    for line in lines:
        draw.text((PADDING, y_offset), line, font=title_font, fill=TEXT_COLOR)
        bbox = draw.textbbox((0, 0), line, font=title_font)
        y_offset += bbox[3] - bbox[1] + 12

    # Author (bottom-left)
    if author:
        author_y = HEIGHT - PADDING - 40
        draw.text((PADDING, author_y), f"By {author}", font=meta_font, fill=MUTED_COLOR)

    # Site name (bottom-right)
    site_name = "yourblog.com"
    bbox = draw.textbbox((0, 0), site_name, font=meta_font)
    site_w = bbox[2] - bbox[0]
    draw.text((WIDTH - PADDING - site_w, HEIGHT - PADDING - 40), site_name, font=meta_font, fill=MUTED_COLOR)

    # Save
    Path(output_path).parent.mkdir(parents=True, exist_ok=True)
    img.save(output_path, "PNG", optimize=True)
    print(f"Saved: {output_path}")


def batch_generate(input_path: str, output_dir: str) -> None:
    """Generate OG images for all posts in a JSON file."""
    with open(input_path) as f:
        posts = json.load(f)

    Path(output_dir).mkdir(parents=True, exist_ok=True)
    total = len(posts)

    for i, post in enumerate(posts, 1):
        title = post.get("title", "Untitled")
        slug = post.get("slug", f"post-{i}")
        category = post.get("category", "")
        author = post.get("author", "")
        output_path = str(Path(output_dir) / f"{slug}.png")

        generate_og_image(title=title, category=category, author=author, output_path=output_path)
        print(f"[{i}/{total}] {slug}.png")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Generate OG images")
    parser.add_argument("--title", help="Post title (single image mode)")
    parser.add_argument("--category", default="", help="Post category")
    parser.add_argument("--author", default="", help="Post author")
    parser.add_argument("--output", default="output/og.png", help="Output path (single mode)")
    parser.add_argument("--input", help="JSON file with posts (batch mode)")
    parser.add_argument("--output-dir", default="output/og/", help="Output directory (batch mode)")
    args = parser.parse_args()

    if args.input:
        batch_generate(args.input, args.output_dir)
    elif args.title:
        generate_og_image(title=args.title, category=args.category, author=args.author, output_path=args.output)
    else:
        parser.print_help()
```

---

## Template: Favicon Set Generator

```python
#!/usr/bin/env python3
"""
Favicon Set Generator
Generates all required favicon sizes from a single source image.

Usage:
  python generate_favicons.py --source assets/logo-512.png --output-dir public/
"""

import argparse
from pathlib import Path
from PIL import Image


FAVICON_SIZES = [
    ("favicon-16x16.png", 16),
    ("favicon-32x32.png", 32),
    ("apple-touch-icon.png", 180),
    ("android-chrome-192x192.png", 192),
    ("android-chrome-512x512.png", 512),
    ("mstile-150x150.png", 150),
]


def generate_favicons(source_path: str, output_dir: str) -> None:
    output = Path(output_dir)
    output.mkdir(parents=True, exist_ok=True)

    source = Image.open(source_path).convert("RGBA")
    print(f"Source image: {source.size[0]}x{source.size[1]} ({source.mode})")

    # Generate individual PNG sizes
    for filename, size in FAVICON_SIZES:
        resized = source.resize((size, size), Image.LANCZOS)
        out_path = output / filename
        resized.save(str(out_path), "PNG")
        print(f"  ✓ {filename} ({size}x{size})")

    # Generate favicon.ico with multiple embedded sizes
    ico_sizes = [(16, 16), (32, 32), (48, 48)]
    ico_images = [source.resize(s, Image.LANCZOS) for s in ico_sizes]
    ico_path = output / "favicon.ico"
    ico_images[0].save(
        str(ico_path),
        format="ICO",
        sizes=ico_sizes,
        append_images=ico_images[1:],
    )
    print(f"  ✓ favicon.ico (16, 32, 48px)")
    print(f"\nAll favicons saved to: {output_dir}")
    print("\nAdd to <head>:")
    print('  <link rel="icon" type="image/x-icon" href="/favicon.ico" />')
    print('  <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />')
    print('  <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />')
    print('  <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />')


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Generate favicon set")
    parser.add_argument("--source", required=True, help="Source PNG (512x512 recommended)")
    parser.add_argument("--output-dir", default="public/", help="Output directory")
    args = parser.parse_args()
    generate_favicons(args.source, args.output_dir)
```

---

## Common Operations

### Load and paste an image with transparency

```python
from PIL import Image

base = Image.new("RGB", (1200, 630), (20, 20, 20))
overlay = Image.open("logo.png").convert("RGBA")

# Resize logo to 60px tall, maintain aspect ratio
target_h = 60
target_w = int(overlay.width * (target_h / overlay.height))
overlay = overlay.resize((target_w, target_h), Image.LANCZOS)

# Paste using alpha channel as mask (third arg = mask)
base.paste(overlay, (80, 80), overlay)
```

### Add semi-transparent color overlay

```python
from PIL import Image

base = Image.open("background.jpg").convert("RGBA")
overlay = Image.new("RGBA", base.size, (0, 0, 0, 160))  # black at 63% opacity
combined = Image.alpha_composite(base, overlay)
result = combined.convert("RGB")  # convert back for JPEG save
```

### Draw a gradient background

```python
from PIL import Image, ImageDraw

img = Image.new("RGB", (1200, 630))
draw = ImageDraw.Draw(img)

# Vertical gradient: top_color → bottom_color
top_color = (30, 20, 60)
bottom_color = (10, 10, 30)

for y in range(630):
    t = y / 630
    r = int(top_color[0] + (bottom_color[0] - top_color[0]) * t)
    g = int(top_color[1] + (bottom_color[1] - top_color[1]) * t)
    b = int(top_color[2] + (bottom_color[2] - top_color[2]) * t)
    draw.line([(0, y), (1200, y)], fill=(r, g, b))
```

### Add text with drop shadow

```python
from PIL import Image, ImageDraw, ImageFont

img = Image.new("RGB", (1200, 630), (20, 20, 20))
draw = ImageDraw.Draw(img)
font = ImageFont.truetype("fonts/Inter-Bold.ttf", 64)

text = "Hello World"
x, y = 80, 200

# Shadow (draw first, behind text)
draw.text((x + 3, y + 3), text, font=font, fill=(0, 0, 0))

# Main text
draw.text((x, y), text, font=font, fill=(255, 255, 255))
```

### Center text horizontally and vertically

```python
def draw_centered(draw, img_width, img_height, text, font, color):
    bbox = draw.textbbox((0, 0), text, font=font)
    text_w = bbox[2] - bbox[0]
    text_h = bbox[3] - bbox[1]
    x = (img_width - text_w) / 2
    y = (img_height - text_h) / 2
    draw.text((x, y), text, font=font, fill=color)
```

### Save with optimization

```python
# PNG (lossless, supports transparency)
img.save("output.png", "PNG", optimize=True)

# JPEG (lossy, smaller, no transparency)
img.convert("RGB").save("output.jpg", "JPEG", quality=85, optimize=True)

# WebP (modern, smaller than both)
img.save("output.webp", "WEBP", quality=85, method=6)
```

---

## HTML/CSS to Image (Alternative to Pillow)

For complex layouts, use html2image or Playwright to render HTML as an image.

### html2image

```bash
pip install html2image
```

```python
from html2image import Html2Image

hti = Html2Image(output_path="output/")
hti.screenshot(
    html_str="""
    <style>
      body { margin: 0; width: 1200px; height: 630px;
             background: #121212; font-family: Inter, sans-serif; }
      .card { padding: 80px; color: white; }
      .title { font-size: 64px; font-weight: bold; line-height: 1.2; }
      .meta { color: #999; font-size: 28px; margin-top: 24px; }
    </style>
    <div class="card">
      <div class="title">My Blog Post Title</div>
      <div class="meta">By Author · Engineering</div>
    </div>
    """,
    save_as="my-post.png",
    size=(1200, 630),
)
```

### Playwright

```bash
pip install playwright
playwright install chromium
```

```python
from playwright.sync_api import sync_playwright

html_content = """<!DOCTYPE html>
<html>
<head>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { width: 1200px; height: 630px; background: #121212;
           font-family: 'Inter', sans-serif; display: flex;
           align-items: center; padding: 80px; }
    h1 { color: white; font-size: 64px; line-height: 1.2; }
  </style>
</head>
<body>
  <h1>My Blog Post Title</h1>
</body>
</html>"""

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(viewport={"width": 1200, "height": 630})
    page.set_content(html_content)
    page.screenshot(path="output/my-post.png", clip={"x": 0, "y": 0, "width": 1200, "height": 630})
    browser.close()
```

**Trade-offs:**

| | html2image | Playwright |
|---|---|---|
| Setup | Simple | Requires browser install |
| Reliability | Varies by OS | Very reliable |
| Font support | System fonts only | System + web fonts |
| Speed | Fast | Slightly slower (browser launch) |
| CI/CD | May need Xvfb on Linux | Works well, needs `playwright install` |

---

## Performance Tips for Large Batches

### Reuse the base template image

```python
# Open once
base_template = Image.open("template.png").convert("RGB")

for post in posts:
    # Copy, don't reopen
    img = base_template.copy()
    draw = ImageDraw.Draw(img)
    # ... draw dynamic content ...
    img.save(f"output/{post['slug']}.png")
```

### Load fonts once

```python
# Outside the loop
title_font = ImageFont.truetype("fonts/Inter-Bold.ttf", 64)
meta_font = ImageFont.truetype("fonts/Inter-Regular.ttf", 28)

for post in posts:
    # Reuse font objects
    draw.text(..., font=title_font, ...)
```

### Multiprocessing for very large batches (1000+)

```python
from multiprocessing import Pool

def generate_one(post):
    # self-contained function
    ...

with Pool(processes=4) as pool:
    pool.map(generate_one, posts)
```

---

## Troubleshooting

**Text appears blurry**
- Use `Image.LANCZOS` for all resizing
- Render at 2x size then downscale: generate at 2400×1260, save at 1200×630

**Font not found in CI/CD**
- Bundle font files in the repo under `fonts/`
- Never rely on system fonts (`/usr/share/fonts/` may differ across environments)

**Logo has white box around it**
- Convert to RGBA: `logo = Image.open("logo.png").convert("RGBA")`
- Use logo as its own paste mask: `base.paste(logo, (x, y), logo)`

**JPEG output has transparency issues**
- JPEG doesn't support alpha; convert before saving: `img.convert("RGB").save(...)`

**Colors look different than expected**
- Check image mode: `print(img.mode)` — should be `RGB` or `RGBA`
- PIL uses `(R, G, B)` tuples, not hex — convert: `tuple(int(h[i:i+2], 16) for i in (1, 3, 5))` from `"#1a1a2e"`

**textsize() is deprecated**
- Use `draw.textbbox((0, 0), text, font=font)` instead; returns `(left, top, right, bottom)`
- Width = `bbox[2] - bbox[0]`, Height = `bbox[3] - bbox[1]`
