---
name: hash-file
description: >
  Compute cryptographic hashes of a file using the QkConvert API. Use when the
  user asks to "hash a file", "get a checksum", "calculate SHA-256", "verify
  file integrity", "get MD5", "compare file checksums", or needs MD5, SHA-1,
  SHA-256, SHA-512, or CRC32.
argument-hint: "<file> [algorithm]"
---

# /hash-file

Compute cryptographic hashes for any file via QkConvert. Costs 1 credit per request.

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

## Workflow

### 1. Parse the request

Identify the file path and which algorithms the user wants. Default to all if not specified. If the user says "checksum" without specifying, use SHA-256 (most common).

### 2. Call the API

Send the file to `/api/v1/file/hash` with field name `file`. Optionally include `options` with an `algorithms` array.

### 3. Present results

Display each hash value with its algorithm name, plus the file size in bytes. Format hashes as monospace text for easy copying.

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `algorithms` | string array | all 5 | Which hashes to compute. Omit for all. |

## Supported Algorithms

| Algorithm | Output length | Use case |
|-----------|--------------|----------|
| `md5` | 32 hex chars | Legacy compatibility, quick deduplication |
| `sha1` | 40 hex chars | Git commit hashes, legacy integrity |
| `sha256` | 64 hex chars | Standard integrity verification |
| `sha512` | 128 hex chars | Higher security requirements |
| `crc32` | 8 hex chars | Fast checksums, ZIP headers, error detection |

## Response Format

```json
{
  "size_bytes": 524288,
  "md5": "d41d8cd98f00b204e9800998ecf8427e",
  "sha1": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
  "sha256": "e3b0c44298fc1c149afbf4c8996fb924...",
  "sha512": "cf83e1357eefb8bdf1542850d66d8007...",
  "crc32": "00000000"
}
```

Only requested algorithms are included in the response. Unrequested ones are omitted.

## Errors

| Code | Meaning | User action |
|------|---------|-------------|
| 400 | Unknown algorithm name or file > 10 MB | Check algorithm spelling and file size |
| 401 | Invalid API key | Check QKCONVERT_API_KEY |
| 403 | Monthly credits exhausted | Upgrade plan or wait for reset |
| 429 | Rate limit exceeded | Wait for Retry-After seconds |

## Examples

Get all hashes for a file:
```
/hash-file document.pdf
```

Get only SHA-256:
```
/hash-file release.zip sha256
```

Compare checksums of two files:
```
/hash-file original.bin sha256
/hash-file download.bin sha256
```

Quick integrity check with CRC32:
```
/hash-file firmware.bin crc32
```
