---
name: convert-image
description: >
  Convert, resize, optimize, or watermark an image using the QkConvert API.
  Use when the user asks to "convert an image", "resize a photo", "compress an
  image", "convert PNG to WebP", "strip EXIF", "add watermark", or needs to
  transform image files. Supports JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG.
argument-hint: "<file> to <format>"
---

# /convert-image

Convert, resize, or process an image file via QkConvert. Costs 1 credit per request.

## Before you start

Check that `QKCONVERT_API_KEY` is set by running `echo $QKCONVERT_API_KEY` in bash. If empty, tell the user:

> Your QkConvert API key is not set. To fix this:
> 1. Sign up free at https://qkconvert.dev/portal/signup
> 2. Create an API key in the dashboard
> 3. Add `export QKCONVERT_API_KEY=sk_live_...` to your shell config (~/.bashrc or ~/.zshrc)
> 4. Restart Claude Code

Do not proceed with the API call until the key is confirmed set.

## Workflow

### 1. Parse the request

Identify the source file, operation, target format, and any options (quality, dimensions, rotation). If no format is specified, suggest webp for photos or png for graphics with transparency.

### 2. Choose the endpoint

| Need | Endpoint | Key parameters |
|------|----------|----------------|
| Change format | `/api/v1/image/convert` | `output_format` (required) |
| Resize, rotate, flip | `/api/v1/image/transform` | `width`, `height`, `rotate`, `flip` |
| Compress for smaller size | `/api/v1/image/optimize` | `quality`, `compression` |
| Add text overlay | `/api/v1/image/watermark` | `text` (required), `position`, `opacity` |
| Get dimensions/format | `/api/v1/image/metadata` | None (returns JSON) |
| Strip EXIF data | `/api/v1/image/exif-strip` | None |
| Multiple thumbnails | `/api/v1/image/thumbnails` | `sizes` array (required) |

### 3. Call the API

Send the file as multipart with field name `image` and options as a JSON string in field `options`. Save the binary response to disk. The output filename should use the target format extension.

### 4. Report

Tell the user: input format, output format, file sizes, and the output path.

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `output_format` | string | convert, transform, watermark, thumbnails | - | Required for convert. One of: png, jpeg, webp, avif, gif, bmp, tiff, ico |
| `quality` | 1-100 | convert, transform, optimize | 85 | Lossy compression for JPEG/WebP/AVIF |
| `compression` | 1-9 | convert, transform, optimize | 6 | PNG compression level (higher = smaller) |
| `width` | integer | transform | - | Target width in px (max 10,000). Aspect ratio preserved if height omitted |
| `height` | integer | transform | - | Target height in px (max 10,000) |
| `rotate` | integer | transform | - | 90, 180, or 270 only |
| `flip` | string | transform | - | "horizontal" or "vertical" |
| `text` | string | watermark | - | Watermark text (max 1,000 chars) |
| `position` | string | watermark | "center" | "center", "top-left", "top-right", "bottom-left", "bottom-right" |
| `opacity` | 0-255 | watermark | 128 | Text opacity |
| `font_size` | float | watermark | 24 | Font size in pixels (1-2000) |
| `color` | string | watermark | "#FFFFFF" | Hex color e.g. "#FF0000" |
| `sizes` | int array | thumbnails | - | List of widths. Max 10 sizes |
| `strip_metadata` | boolean | convert, transform, optimize | false | Remove EXIF/ICC data |

## Formats

**Input:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG
**Output:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO

- SVG is input-only (rasterized on conversion)
- ICO output auto-clamps to 256x256 max

## Errors

| Code | Meaning | User action |
|------|---------|-------------|
| 400 | Unsupported format, corrupt file, missing parameter, or file > 10 MB | Check format and file size |
| 401 | Invalid API key | Check QKCONVERT_API_KEY |
| 403 | Monthly credits exhausted (free tier hard cap) | Upgrade plan or wait for reset |
| 429 | Rate limit exceeded | Wait for Retry-After seconds |

## Examples

Convert a PNG to WebP with quality control:
```
/convert-image banner.png to webp at quality 90
```

Resize to 800px wide and change format:
```
/convert-image photo.jpg resize to 800px wide, output as webp
```

Generate thumbnails at multiple sizes:
```
/convert-image product.jpg generate thumbnails at 64, 128, 256, 512
```

Strip metadata for privacy before sharing:
```
/convert-image selfie.jpg strip all metadata
```

Add a watermark:
```
/convert-image photo.jpg add watermark "© 2026 My Company" in bottom-right, opacity 180
```
