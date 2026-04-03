---
name: process-document
description: >
  Process PDF documents using the QkConvert API. Use when the user asks to
  "merge PDFs", "split a PDF", "compress a PDF", "extract text from PDF",
  "convert PDF to images", "add watermark to PDF", or "create a PDF from images".
argument-hint: "<file> [operation]"
---

# /process-document

Process PDF files via QkConvert.

## Workflow

### 1. Parse the request

Identify:
- **Source file(s)** — path to PDF(s) or images
- **Operation** — merge, split, compress, watermark, text extract, to-image, from-images, text-to-pdf

### 2. Choose the endpoint

| Need | Endpoint |
|------|----------|
| Combine PDFs | `/api/v1/doc/merge` |
| Extract pages | `/api/v1/doc/split` |
| Reduce file size | `/api/v1/doc/compress` |
| Add text overlay | `/api/v1/doc/watermark` |
| Get text content | `/api/v1/doc/text` |
| Render pages as images | `/api/v1/doc/to-image` |
| Create PDF from images | `/api/v1/doc/from-images` |
| Create PDF from text | `/api/v1/doc/text-to-pdf` |
| Get page count/metadata | `/api/v1/doc/metadata` |

### 3. Call the API

Send files as multipart form data. For merge and from-images, send multiple files. Save the response to disk.

### 4. Report

Show: operation, page count, file sizes, output path.

## Notes

- Split uses page ranges like "1-5,8,10-12"
- to-image returns a ZIP if multiple pages, single image for one page
- Each operation costs 2 credits
