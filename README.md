# QkConvert Plugin for Claude

File conversion API plugin — convert images, audio, documents, and data formats directly from Claude.

## Setup

1. Sign up at [qkconvert.dev](https://qkconvert.dev/portal/signup) and create an API key
2. Set your API key as an environment variable:
   ```
   export QKCONVERT_API_KEY=sk_live_...
   ```
3. Install the plugin:
   ```
   claude plugin install qkconvert
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

## Pricing

Free tier: 500 credits/month. Paid plans from $6/month.
See [qkconvert.dev](https://qkconvert.dev/#pricing) for details.

## Links

- [API Docs](https://qkconvert.dev/docs)
- [Sandbox](https://qkconvert.dev/sandbox) — try without signing up
- [OpenAPI Spec](https://qkconvert.dev/api/openapi.json)
