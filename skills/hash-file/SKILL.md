---
name: hash-file
description: >
  Compute cryptographic hashes of a file using the QkConvert API. Use when the
  user asks to "hash a file", "get a checksum", "calculate SHA-256", "verify
  file integrity", "get MD5", or needs MD5, SHA-1, SHA-256, SHA-512, or CRC32.
argument-hint: "<file> [algorithm]"
---

# /hash-file

Compute cryptographic hashes for any file via QkConvert.

## Workflow

### 1. Parse the request

Identify:
- **File** — path to any file
- **Algorithms** — which hashes to compute (default: all)

### 2. Call the API

Send the file to `/api/v1/file/hash` with the algorithms option.

### 3. Present results

Display each hash value with its algorithm name, plus the file size.

## Supported Algorithms

- `md5` — 128-bit, legacy compatibility
- `sha1` — 160-bit, used by Git
- `sha256` — 256-bit, standard integrity check
- `sha512` — 512-bit, higher security
- `crc32` — 32-bit, fast checksum

## Notes

- Defaults to all algorithms if none specified
- Each operation costs 1 credit
