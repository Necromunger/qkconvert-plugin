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

Convert, resize, or process an image file via QkConvert.

## Workflow

### 1. Parse the request

Identify:
- **Source file** — path to the image
- **Operation** — convert, resize, optimize, watermark, or metadata
- **Target format** — png, jpeg, webp, avif, gif, bmp, tiff, ico (default: webp)
- **Options** — quality (1-100), width/height, rotation, flip

### 2. Choose the endpoint

| Need | Endpoint |
|------|----------|
| Change format | `/api/v1/image/convert` |
| Resize, rotate, flip | `/api/v1/image/transform` |
| Compress for smaller size | `/api/v1/image/optimize` |
| Add text overlay | `/api/v1/image/watermark` |
| Get dimensions/format info | `/api/v1/image/metadata` |
| Strip EXIF data | `/api/v1/image/exif-strip` |
| Multiple thumbnail sizes | `/api/v1/image/thumbnails` |

### 3. Call the API

Send the file as multipart form data with the options JSON. Save the binary response to disk.

### 4. Report

Show: source format, output format, input size, output size, output path.

## Supported Formats

Input: JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG
Output: JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO

## Notes

- ICO output is clamped to 256x256 max automatically
- SVG is input-only (rasterized to the target format)
- WebP typically gives the best size-to-quality ratio for photos
- Each operation costs 1 credit
