---
name: process-document
description: >
  Process PDF documents using the QkConvert API. Use when the user asks to
  "merge PDFs", "split a PDF", "compress a PDF", "extract text from PDF",
  "convert PDF to images", "add watermark to PDF", "create PDF from images",
  or "convert text to PDF".
argument-hint: "<file> [operation]"
---

# /process-document

Process PDF files via QkConvert. Costs 2 credits per request.

## Before you start

Check that `QKCONVERT_API_KEY` is set by running `test -n "$QKCONVERT_API_KEY" && echo "set" || echo "not set"` in bash. If not set, tell the user:

> Your QkConvert API key is not set. To fix this:
> 1. Sign up free at https://qkconvert.dev/portal/signup
> 2. Create an API key in the dashboard
> 3. Set the env var permanently:
>    - **Mac/Linux:** Add `export QKCONVERT_API_KEY=sk_live_...` to `~/.bashrc` or `~/.zshrc`
>    - **Windows:** Run `setx QKCONVERT_API_KEY sk_live_...` in CMD or PowerShell
> 4. Restart Claude Code

Do not proceed until the key is confirmed set.

## How to call the API

**Do not call the MCP tools directly** — the OpenAPI-to-MCP bridge cannot send multipart/form-data, so all tool calls arrive with empty content. Always use `curl` via the Bash tool:

```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/{endpoint} \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@{input_file}" \
  -F "options={json_options}" \
  -o {output_file}
```

**Rules:**
- Always use `-s` (silent)
- Always use `-o` for binary output (PDF, image, ZIP)
- Field name is `file` for document endpoints
- Use escaped double quotes: `-F "options={\"pages\":\"1-5\"}"`
- For merge and from-images, send multiple `-F "file=@..."` fields

## Curl commands by operation

### Merge PDFs
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/merge \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@part1.pdf" \
  -F "file=@part2.pdf" \
  -F "file=@part3.pdf" \
  -o merged.pdf
```

### Split (extract pages)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/split \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf" \
  -F "options={\"pages\":\"1-5,8,10-12\"}" \
  -o extracted.pdf
```

### Compress
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/compress \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@large.pdf" \
  -o compressed.pdf
```

### Watermark
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/watermark \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf" \
  -F "options={\"text\":\"DRAFT\",\"opacity\":0.3,\"font_size\":48}" \
  -o watermarked.pdf
```

### Extract text (returns JSON, no -o)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/text \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf"
```

### Render to images (ZIP for multiple pages)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/to-image \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf" \
  -F "options={\"output_format\":\"png\",\"dpi\":300,\"pages\":\"1-3\"}" \
  -o pages.zip
```

### Create PDF from images
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/from-images \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@scan1.jpg" \
  -F "file=@scan2.jpg" \
  -F "file=@scan3.jpg" \
  -o scanned.pdf
```

### Create PDF from text
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/text-to-pdf \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.txt" \
  -F "options={\"font_size\":12,\"page_size\":\"a4\"}" \
  -o document.pdf
```

### Metadata (returns JSON, no -o)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/doc/metadata \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf"
```

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `pages` | string | split, to-image | all | "1-5", "2,4,6", "1-3,8" (1-based) |
| `text` | string | watermark | - | Max 500 chars |
| `opacity` | 0.0-1.0 | watermark | 0.3 | Transparency |
| `font_size` | 1-500 | watermark, text-to-pdf | 48/12 | Points |
| `color` | string | watermark | "#888888" | Hex color |
| `rotation` | float | watermark | 45 | Degrees |
| `dpi` | 72-600 | to-image | 150 | Render resolution |
| `output_format` | string | to-image | "png" | "png" or "jpeg" |
| `page_size` | string | text-to-pdf | "a4" | "a4", "letter", "legal" |

## Edge cases

- **to-image**: returns single image for 1 page, ZIP for multiple
- **text**: non-OCR (embedded text only, not scanned docs)
- **merge**: 2-50 files max
- **from-images**: 1-50 images, accepts any image format
- Max file size: 10 MB per file

## After the call

Report: operation, page count (if applicable), file sizes, output path.
