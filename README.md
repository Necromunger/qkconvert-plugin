# QkConvert Plugin for Claude

File conversion API plugin — convert images, audio, documents, and data formats directly from Claude.

## Install

**1. Add the marketplace:**
```
/plugin marketplace add Necromunger/qkconvert-plugin
```

**2. Install the plugin:**
```
/plugin install qkconvert@qkconvert-plugin
```

**3. Set your API key** (sign up free at [qkconvert.dev](https://qkconvert.dev/portal/signup)):
```
export QKCONVERT_API_KEY=sk_live_...
```

## Skills

| Skill | Description |
|-------|-------------|
| `/convert-image` | Convert, resize, optimize, watermark images (1 credit) |
| `/convert-audio` | Convert, trim, merge, split, normalize audio (3 credits) |
| `/process-document` | Merge, split, compress, watermark PDFs (2 credits) |
| `/convert-data` | Convert between CSV, JSON, XML, YAML, XLSX, TOML (1 credit) |
| `/hash-file` | Compute MD5, SHA-1, SHA-256, SHA-512, CRC32 (1 credit) |

## Examples

```
/convert-image photo.png to webp
/convert-audio podcast.wav to mp3
/process-document merge report1.pdf report2.pdf
/convert-data users.csv to json
/hash-file document.pdf sha256
```

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
        "API_HEADERS": "Authorization:Bearer sk_live_..."
      }
    }
  }
}
```

## Pricing

Free tier: 500 credits/month. Paid plans from $6/month.
See [qkconvert.dev](https://qkconvert.dev/#pricing) for details.

## Links

- [API Docs](https://qkconvert.dev/docs)
- [Sandbox](https://qkconvert.dev/sandbox) — try without signing up
- [OpenAPI Spec](https://qkconvert.dev/api/openapi.json)
