---
name: archive
description: >
  Create ZIP archives or inspect ZIP contents using the QkConvert API.
  Use when the user asks to "zip files", "create a ZIP", "archive files",
  "inspect a ZIP", "list ZIP contents", or needs to bundle files into a ZIP.
argument-hint: "<files...>"
---

# /archive

Create ZIP archives or inspect ZIP contents via QkConvert. Costs 2 credits per request.

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

**Do not call the MCP tools directly** - the OpenAPI-to-MCP bridge cannot send multipart/form-data, so all tool calls arrive with empty content. Always use `curl` via the Bash tool.

## Curl commands by operation

### Create ZIP (multiple files)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/archive/zip \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@document.pdf" \
  -F "file=@photo.jpg" \
  -F "file=@data.csv" \
  -o archive.zip
```

**Rules:**
- Always use `-s` (silent)
- Always use `-o` to save the ZIP output
- Field name is `file` - send multiple `-F "file=@..."` for each file
- Original filenames are preserved inside the ZIP
- Maximum 100 files per request
- Maximum 50 MB total uncompressed size

### Inspect ZIP (returns JSON, no -o needed)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/archive/zip-info \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@archive.zip"
```

Returns:
```json
{
  "file_count": 3,
  "total_size": 1048576,
  "files": [
    {"filename": "document.pdf", "size": 524288},
    {"filename": "photo.jpg", "size": 412000},
    {"filename": "data.csv", "size": 112288}
  ]
}
```

## Limits

- **Max files per ZIP:** 100
- **Max total uncompressed size:** 50 MB
- **Max upload size:** 10 MB per request

## After the call

For ZIP creation: report the output path, number of files, and output file size using `wc -c < file`.
For ZIP info: display the file listing with sizes formatted readably.
