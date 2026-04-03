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

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `output_format` | string | convert, transform, watermark, thumbnails | - | png, jpeg, webp, avif, gif, bmp, tiff, ico |
| `quality` | 1-100 | convert, transform, optimize | 85 | Lossy compression for JPEG/WebP/AVIF |
| `compression` | 1-9 | convert, transform, optimize | 6 | PNG compression level |
| `width` | integer | transform | - | Max 10,000. Aspect ratio preserved if height omitted |
| `height` | integer | transform | - | Max 10,000 |
| `rotate` | integer | transform | - | 90, 180, or 270 only |
| `flip` | string | transform | - | "horizontal" or "vertical" |
| `text` | string | watermark | - | Max 1,000 chars |
| `position` | string | watermark | "center" | "center", "top-left", "top-right", "bottom-left", "bottom-right" |
| `opacity` | 0-255 | watermark | 128 | Text opacity |
| `sizes` | int array | thumbnails | - | List of widths. Max 10 |
| `strip_metadata` | boolean | convert, transform, optimize | false | Remove EXIF/ICC data |

## Formats

**Input:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG
**Output:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO (SVG input-only, ICO max 256x256)

## After the call

Report to the user: input file, output file, and file sizes. Use `wc -c < file` to get sizes.
