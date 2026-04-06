---
name: convert-image
description: >
  Convert, resize, optimize, crop, watermark, or batch-process images using the
  QkConvert API. Use when the user asks to "convert an image", "resize a photo",
  "compress an image", "convert PNG to WebP", "strip EXIF", "add watermark",
  "crop an image", "generate blurhash", "OCR an image", "create a GIF",
  "extract GIF frames", "reverse a GIF", "batch convert", "generate QR code",
  "create barcode", "make QR", or needs to transform image files.
  Supports JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG.
argument-hint: "<file> to <format>"
---

# /convert-image

Convert, resize, or process an image file via QkConvert. Costs 1 credit per request.

## Before you start

Check that `QKCONVERT_API_KEY` is set by running `test -n "$QKCONVERT_API_KEY" && echo "set" || echo "not set"` in bash. If not set, tell the user:

> Your QkConvert API key is not set. To fix this:
> 1. Sign up free at https://qkconvert.dev/portal/signup
> 2. Create an API key in the dashboard
> 3. Set the env var permanently:
>    - **Mac/Linux:** Add `export QKCONVERT_API_KEY=sk_live_...` to `~/.bashrc` or `~/.zshrc`
>    - **Windows:** Run `setx QKCONVERT_API_KEY sk_live_...` in CMD or PowerShell
> 4. Restart Claude Code

Do not proceed with the API call until the key is confirmed set.

## How to call the API

**Do not call the MCP tools directly** — the OpenAPI-to-MCP bridge cannot send multipart/form-data, so all tool calls arrive with empty content. Always use `curl` via the Bash tool.

Every image endpoint uses the same pattern:

```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/{endpoint} \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@{input_file}" \
  -F "options={json_options}" \
  -o {output_file}
```

**Rules:**
- Always use `-s` (silent) to suppress progress bars
- Always use `-o` to save binary output to a file
- Options JSON must use escaped double quotes: `-F "options={\"output_format\":\"webp\"}"`
- Do NOT use single quotes around the options value — use escaped doubles
- The output filename should match the target format extension

## Curl commands by operation

### Convert format
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/convert \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg" \
  -F "options={\"output_format\":\"webp\",\"quality\":85}" \
  -o photo.webp
```

### Resize / rotate / flip
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/transform \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg" \
  -F "options={\"width\":800,\"output_format\":\"webp\"}" \
  -o photo_800.webp
```

### Optimize (compress)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/optimize \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg" \
  -F "options={\"quality\":75}" \
  -o photo_optimized.jpg
```

### Watermark
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/watermark \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg" \
  -F "options={\"text\":\"Copyright 2026\",\"position\":\"bottom-right\",\"opacity\":180}" \
  -o photo_watermarked.jpg
```

### Metadata (returns JSON, no -o needed)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/metadata \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg"
```

### Strip EXIF
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/exif-strip \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg" \
  -o photo_clean.jpg
```

### Thumbnails (returns ZIP for multiple sizes)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/thumbnails \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo.jpg" \
  -F "options={\"sizes\":[64,128,256,512],\"output_format\":\"webp\"}" \
  -o thumbnails.zip
```

### Crop (pixel or aspect ratio)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/crop \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@{input_file}" \
  -F "options={\"x\":100,\"y\":50,\"width\":800,\"height\":600}" \
  -o cropped.jpg
```
Or aspect-ratio mode:
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/crop \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@{input_file}" \
  -F "options={\"aspect_ratio\":\"16:9\",\"gravity\":\"center\",\"output_format\":\"webp\"}" \
  -o cropped.webp
```

### Blurhash encode (returns JSON, no -o needed)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/blurhash \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@{input_file}" \
  -F "options={\"components_x\":4,\"components_y\":3}"
```
Returns: `{"blurhash":"LEHV6n...","width":800,"height":600,"components_x":4,"components_y":3}`

### Blurhash decode (JSON body, not multipart)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/blurhash-decode \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"blurhash\":\"LEHV6nWB2yk8pyo0adR*.7kCMdnj\",\"width\":200,\"height\":150}" \
  -o placeholder.png
```

### OCR (returns JSON, no -o needed)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/ocr \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@{input_file}" \
  -F "options={\"language\":\"eng\"}"
```
Returns: `{"text":"extracted text...","char_count":42}`

### GIF create (multiple files)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/gif-create \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@frame1.png" \
  -F "image=@frame2.png" \
  -F "image=@frame3.png" \
  -F "options={\"frame_delay_ms\":200,\"loop_count\":0}" \
  -o animation.gif
```

### GIF extract
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/gif-extract \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@animation.gif" \
  -F "options={\"frame\":\"0\",\"output_format\":\"png\"}" \
  -o frame0.png
```
For all frames: `options={"frame":"all"}` returns a ZIP.

### GIF reverse
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/gif-reverse \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@animation.gif" \
  -o reversed.gif
```

### Convert batch (returns ZIP)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/convert-batch \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo1.jpg" \
  -F "image=@photo2.png" \
  -F "image=@photo3.bmp" \
  -F "options={\"output_format\":\"webp\",\"quality\":85}" \
  -o converted.zip
```

### QR code (no file upload - options only)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/qr \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "options={\"content\":\"https://example.com\",\"size\":512,\"output_format\":\"png\"}" \
  -o qr.png
```
SVG output:
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/qr \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "options={\"content\":\"https://example.com\",\"output_format\":\"svg\"}"
```
SVG returns JSON: `{"svg":"<svg ...>","width":256,"height":256}`. Save the `svg` field to a file.

### Barcode (no file upload - options only)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/barcode \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "options={\"content\":\"1234567890128\",\"format\":\"ean13\",\"output_format\":\"png\",\"width\":300,\"height\":100}" \
  -o barcode.png
```
Supported formats: code128, code39, ean13, ean8, upca, itf, codabar.

### Transform batch (returns ZIP)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/image/transform-batch \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "image=@photo1.jpg" \
  -F "image=@photo2.jpg" \
  -F "options={\"width\":800,\"output_format\":\"webp\"}" \
  -o resized.zip
```

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `output_format` | string | convert, transform, watermark, thumbnails, crop, gif-extract, blurhash-decode, convert-batch, transform-batch | - | png, jpeg, webp, avif, gif, bmp, tiff, ico |
| `quality` | 1-100 | convert, transform, optimize, convert-batch, transform-batch | 85 | Lossy compression for JPEG/WebP/AVIF |
| `compression` | 1-9 | convert, transform, optimize, convert-batch, transform-batch | 6 | PNG compression level |
| `width` | integer | transform, blurhash-decode, transform-batch | - | Max 10,000. Aspect ratio preserved if height omitted |
| `height` | integer | transform, blurhash-decode, transform-batch | - | Max 10,000 |
| `rotate` | integer | transform, transform-batch | - | 90, 180, or 270 only |
| `flip` | string | transform, transform-batch | - | "horizontal" or "vertical" |
| `text` | string | watermark | - | Max 1,000 chars |
| `position` | string | watermark | "center" | "center", "top-left", "top-right", "bottom-left", "bottom-right" |
| `opacity` | 0-255 | watermark | 128 | Text opacity |
| `sizes` | int array | thumbnails | - | List of widths. Max 10 |
| `strip_metadata` | boolean | convert, transform, optimize | false | Remove EXIF/ICC data |
| `x` | integer | crop | - | Left edge of crop region (pixels) |
| `y` | integer | crop | - | Top edge of crop region (pixels) |
| `aspect_ratio` | string | crop | - | e.g. "16:9", "4:3", "1:1". Alternative to pixel crop |
| `gravity` | string | crop | "center" | "center", "top", "bottom", "left", "right", "top-left", "top-right", "bottom-left", "bottom-right" |
| `components_x` | 1-9 | blurhash | 4 | Horizontal blurhash components |
| `components_y` | 1-9 | blurhash | 3 | Vertical blurhash components |
| `blurhash` | string | blurhash-decode | - | Required. The blurhash string to decode |
| `punch` | float | blurhash-decode | 1.0 | Contrast adjustment |
| `language` | string | ocr | "eng" | Tesseract language code |
| `frame_delay_ms` | integer | gif-create | 100 | Delay between frames in milliseconds |
| `loop_count` | integer | gif-create | 0 | Number of loops (0 = infinite) |
| `frame` | string | gif-extract | - | Frame index ("0", "1", ...) or "all" for ZIP |
| `content` | string | qr, barcode | - | Required. Data to encode |
| `size` | integer | qr | 256 | QR image size in pixels (width = height) |
| `fg_color` | string | qr | "#000000" | Foreground hex color |
| `bg_color` | string | qr | "#FFFFFF" | Background hex color |
| `format` | string | barcode | - | Required. code128, code39, ean13, ean8, upca, itf, codabar |
| `width` | integer | barcode | 300 | Output width in pixels |
| `height` | integer | barcode | 100 | Output height in pixels |

## Formats

**Input:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG
**Output:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO (SVG input-only, ICO max 256x256)

## After the call

Report to the user: input file, output file, and file sizes. Use `wc -c < file` to get sizes.
