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

## Workflow

### 1. Parse the request

Identify the source file(s), operation, and any options (page range, watermark text, DPI, output format).

### 2. Choose the endpoint

| Need | Endpoint | Key parameters |
|------|----------|----------------|
| Combine PDFs | `/api/v1/doc/merge` | 2-50 PDF files |
| Extract pages | `/api/v1/doc/split` | `pages` range string |
| Reduce file size | `/api/v1/doc/compress` | None |
| Add text watermark | `/api/v1/doc/watermark` | `text` (required) |
| Get text content | `/api/v1/doc/text` | None (returns JSON) |
| Render as images | `/api/v1/doc/to-image` | `pages`, `dpi`, `output_format` |
| Create from images | `/api/v1/doc/from-images` | 1-50 image files |
| Create from text | `/api/v1/doc/text-to-pdf` | Text/markdown file |
| Get page count/info | `/api/v1/doc/metadata` | None (returns JSON) |

### 3. Call the API

Send files with field name `file` or `document`. For merge and from-images, send multiple `file` fields. Save binary response to disk.

### 4. Report

Tell the user: operation, page count, file sizes, output path.

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `pages` | string | split, to-image | all | Page range: "1-5", "2,4,6", "1-3,8,10-12" (1-based) |
| `text` | string | watermark | - | Required. Watermark text (max 500 chars) |
| `opacity` | float 0-1 | watermark | 0.3 | Watermark transparency |
| `font_size` | float 1-500 | watermark | 48 | Font size in points |
| `color` | string | watermark | "#888888" | Hex color |
| `rotation` | float | watermark | 45 | Rotation in degrees |
| `dpi` | integer 72-600 | to-image | 150 | Render resolution |
| `output_format` | string | to-image | "png" | "png" or "jpeg" |
| `font_size` | float 1-500 | text-to-pdf | 12 | Text font size |
| `page_size` | string | text-to-pdf | "a4" | "a4", "letter", or "legal" |
| `title` | string | text-to-pdf | - | PDF title metadata (max 512 chars) |

## Edge Cases

- **to-image** returns a single image for 1 page, or a ZIP archive for multiple pages
- **from-images** accepts JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG — images are JPEG-encoded at 90% quality in the PDF
- **text** extraction is non-OCR (embedded text only, not scanned documents)
- **merge** accepts 2-50 PDF files
- Max file size: 10 MB per file

## Errors

| Code | Meaning | User action |
|------|---------|-------------|
| 400 | Corrupt PDF, invalid page range, too many files, or file > 10 MB | Check input and parameters |
| 401 | Invalid API key | Check QKCONVERT_API_KEY |
| 403 | Monthly credits exhausted | Upgrade plan or wait for reset |
| 429 | Rate limit exceeded | Wait for Retry-After seconds |

## Examples

Merge three PDFs:
```
/process-document merge part1.pdf part2.pdf part3.pdf
```

Extract pages 1-5 and page 8:
```
/process-document split report.pdf pages 1-5,8
```

Compress a large PDF:
```
/process-document compress thesis.pdf
```

Render PDF pages as images at 300 DPI:
```
/process-document render contract.pdf as jpeg at 300 dpi
```

Create a PDF from images:
```
/process-document create pdf from scan1.jpg scan2.jpg scan3.jpg
```
