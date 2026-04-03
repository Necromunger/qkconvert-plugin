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

Do not proceed until the key is confirmed set.

## How to call the API

**Do not call the MCP tools directly** — the OpenAPI-to-MCP bridge cannot send multipart/form-data, so all tool calls arrive with empty content. Always use `curl` via the Bash tool. This endpoint returns JSON (no binary output).

### All algorithms (default)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/file/hash \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf"
```

### Specific algorithms
```bash
curl -s -X POST https://qkconvert.dev/api/v1/file/hash \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf" \
  -F "options={\"algorithms\":[\"sha256\",\"md5\"]}"
```

### Single algorithm
```bash
curl -s -X POST https://qkconvert.dev/api/v1/file/hash \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@release.zip" \
  -F "options={\"algorithms\":[\"sha256\"]}"
```

**Rules:**
- Always use `-s` (silent)
- No `-o` needed — response is JSON printed to stdout
- Field name is `file`
- Use escaped double quotes: `-F "options={\"algorithms\":[\"sha256\"]}"`

## Response format

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

Only requested algorithms appear. Unrequested ones are omitted.

## Supported algorithms

| Algorithm | Output | Use case |
|-----------|--------|----------|
| `md5` | 32 hex chars | Legacy deduplication |
| `sha1` | 40 hex chars | Git, legacy integrity |
| `sha256` | 64 hex chars | Standard integrity check |
| `sha512` | 128 hex chars | Higher security |
| `crc32` | 8 hex chars | Fast checksums, ZIP headers |

If the user doesn't specify an algorithm, return all. If they say "checksum" without specifying, use sha256.

## After the call

Parse the JSON response and display each hash with its algorithm name. Format hashes as monospace for easy copying. Include the file size.
