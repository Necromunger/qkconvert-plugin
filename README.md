# QkConvert Plugin for Claude

File conversion API plugin — convert images, audio, documents, and data formats directly from Claude Code and Cowork. [Try it in the sandbox](https://qkconvert.dev/sandbox) — 20 requests/day, no signup required.

## Install

**1. Add the marketplace:**
```
/plugin marketplace add Necromunger/qkconvert-plugin
```

**2. Install the plugin:**
```
/plugin install qkconvert@qkconvert-plugin
```

**3. Set your API key:**

Sign up free at [qkconvert.dev](https://qkconvert.dev/portal/signup) (Google, GitHub, or email), then create an API key in the dashboard.

**Claude Code (Mac/Linux)** — add to your shell config (`.bashrc`, `.zshrc`):
```bash
export QKCONVERT_API_KEY=sk_live_...
```

**Claude Code (Windows)** — run in CMD or PowerShell (persists across sessions):
```
setx QKCONVERT_API_KEY sk_live_...
```

**Cowork** — set in the plugin's environment configuration when prompted during install, or via Settings > Plugins > QkConvert > Environment Variables.

The plugin reads `QKCONVERT_API_KEY` from the environment and passes it as `Authorization: Bearer` to all API calls.

> **No key yet?** Try the [sandbox](https://qkconvert.dev/sandbox) first — 20 free requests per day with no signup.

## Skills

6 skills covering all 64 API endpoints. All endpoints **must** be called via curl (as the skills do), not by invoking the MCP tools directly - the OpenAPI-to-MCP bridge cannot transmit multipart/form-data bodies, so direct tool calls arrive with empty content regardless of whether you pass a file path or inline data. The MCP tools are exposed for discovery and documentation only.

| Skill | What it does | Credits | Endpoints covered |
|-------|-------------|---------|-------------------|
| `/convert-image` | Convert, resize, optimize, watermark, strip EXIF, thumbnails, QR codes, barcodes, OCR, batch ops, blurhash, GIF, crop | 1 | 18 |
| `/convert-audio` | Convert, trim, merge, split, normalize volume, generate waveform | 3 | 7 |
| `/process-document` | Merge, split, compress, watermark, extract text, render to image, create PDF, protect, office-to-pdf | 2 | 11 |
| `/convert-data` | Convert between CSV, JSON, XML, YAML, XLSX, TOML + Markdown to HTML | 1 | 25 |
| `/hash-file` | Compute MD5, SHA-1, SHA-256, SHA-512, CRC32 for any file | 1 | 1 |
| `/archive` | Create ZIP archives, inspect ZIP contents | 2 | 2 |

**Total: 64 endpoints across 7 categories** - see the [OpenAPI spec](https://qkconvert.dev/api/openapi.json) for the full inventory.

## Output Files

All skills save output files to the **current working directory** by default, using the original filename with the new extension (e.g. `photo.png` → `photo.webp`). If a file with that name already exists, the skill will ask before overwriting.

Skills are **manually chainable** — run one, then reference the output path in the next:
```
/convert-image photo.tiff to png
/hash-file photo.png sha256
```

Automatic piping between skills is not currently supported — each skill runs as an independent API call.

## Examples

**Image conversion with options:**
```
/convert-image photo.png to webp at quality 85, resize to 800px wide
```

**Trim an audio clip:**
```
/convert-audio podcast.mp3 trim from 1:30 to 5:45, output as wav
```

**Merge PDFs:**
```
/process-document merge chapter1.pdf chapter2.pdf chapter3.pdf
```

**Convert a spreadsheet to JSON:**
```
/convert-data sales.xlsx to json, pretty-printed
```

**Verify file integrity:**
```
/hash-file release.zip sha256
```

## Pricing

| Plan | Monthly | Credits | Rate limit | Overage |
|------|---------|---------|------------|---------|
| Free | $0 | 500/mo | 5 RPM | Hard cap |
| Hobby | $6 | 1,000/mo | 30 RPM | $0.006/credit |
| Starter | $15 | 5,000/mo | 60 RPM | $0.003/credit |
| Pro | $60 | 50,000/mo | 300 RPM | $0.002/credit |

**Credit costs by category:**

| Category | Credits | Endpoints |
|----------|---------|-----------|
| Images (convert, resize, optimize, watermark, QR, barcode, EXIF, thumbnails, metadata, OCR, batch ops, blurhash, GIF, crop) | 1 | 18 |
| Data (CSV, JSON, XML, YAML, XLSX, TOML, Markdown — 25 conversion pairs) | 1 | 25 |
| File utilities (hash/checksum) | 1 | 1 |
| Documents (merge, split, compress, watermark, text, to-image, from-images, text-to-pdf, metadata, protect, office-to-pdf) | 2 | 11 |
| Archives (ZIP create, ZIP inspect) | 2 | 2 |
| Audio (convert, trim, merge, split, normalize, waveform, metadata) | 3 | 7 |

Free tier: 500 credits total per month across all categories. For example, 500 image conversions, or 166 audio conversions, or any mix.

## Limits

- **Max file size:** 10 MB (authenticated), 2 MB (sandbox)
- **Supported image input:** JPEG, PNG, WebP, AVIF, GIF, BMP, TIFF, ICO, SVG
- **Supported audio input:** MP3, WAV, FLAC, OGG, AAC, AIFF
- **Audio output:** MP3, WAV, FLAC only
- **Documents:** PDF only
- **Archives:** ZIP only

## Error Reference

| HTTP Code | Error code | Meaning |
|-----------|-----------|---------|
| 400 | `bad_request` | Malformed input, unsupported format, missing parameter, or file too large |
| 401 | `unauthorized` | API key missing or invalid — check `QKCONVERT_API_KEY` |
| 403 | `quota_exceeded` | Monthly credits exhausted (free tier). Upgrade or wait for billing reset |
| 429 | `rate_limited` | Too many requests — wait for `Retry-After` header value |
| 500 | `internal_error` | Server error (rare). Retry the request |

Error responses are JSON: `{"error": {"code": "bad_request", "message": "Unsupported format: 'xyz'"}}`

## Manual MCP Config

If you prefer to configure MCP directly instead of using the plugin:

```json
{
  "mcpServers": {
    "qkconvert": {
      "command": "npx",
      "args": ["-y", "@ivotoby/openapi-mcp-server"],
      "env": {
        "API_BASE_URL": "https://qkconvert.dev",
        "OPENAPI_SPEC_PATH": "https://qkconvert.dev/api/openapi.json",
        "API_HEADERS": "Authorization:Bearer YOUR_API_KEY"
      }
    }
  }
}
```

## Plugin Structure

```
.claude-plugin/
  plugin.json           # Plugin manifest (name, version, description)
  marketplace.json      # Marketplace catalog for discovery
.mcp.json               # MCP server config (OpenAPI bridge)
skills/
  convert-image/SKILL.md
  convert-audio/SKILL.md
  process-document/SKILL.md
  convert-data/SKILL.md
  hash-file/SKILL.md
  archive/SKILL.md
```

## Links

- [Website](https://qkconvert.dev)
- [API Docs](https://qkconvert.dev/docs) (interactive Swagger UI)
- [Sandbox](https://qkconvert.dev/sandbox) — try 20 requests/day without signing up
- [OpenAPI Spec](https://qkconvert.dev/api/openapi.json)
- [llms.txt](https://qkconvert.dev/llms.txt) — AI-readable API reference

## License

MIT — see [LICENSE](LICENSE).
