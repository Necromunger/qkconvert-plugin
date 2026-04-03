---
name: convert-audio
description: >
  Convert, trim, merge, split, or normalize audio using the QkConvert API.
  Use when the user asks to "convert audio", "trim a clip", "merge audio files",
  "normalize volume", "generate a waveform", or needs to process MP3, WAV, FLAC,
  OGG, AAC, or AIFF files.
argument-hint: "<file> to <format>"
---

# /convert-audio

Process audio files via QkConvert. Costs 3 credits per request.

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
curl -s -X POST https://qkconvert.dev/api/v1/audio/{endpoint} \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@{input_file}" \
  -F "options={json_options}" \
  -o {output_file}
```

**Rules:**
- Always use `-s` (silent)
- Always use `-o` to save binary output
- Use escaped double quotes in options: `-F "options={\"output_format\":\"mp3\"}"`
- Field name is `audio` (not `file`) for audio endpoints

## Curl commands by operation

### Convert format
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/convert \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@recording.wav" \
  -F "options={\"output_format\":\"mp3\",\"bitrate\":192}" \
  -o recording.mp3
```

### Trim
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/trim \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@podcast.mp3" \
  -F "options={\"start_secs\":30.0,\"end_secs\":90.0,\"output_format\":\"mp3\"}" \
  -o clip.mp3
```

### Merge (multiple files)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/merge \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@intro.mp3" \
  -F "audio=@main.mp3" \
  -F "audio=@outro.mp3" \
  -F "options={\"output_format\":\"mp3\"}" \
  -o merged.mp3
```

### Split
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/split \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@long.mp3" \
  -F "options={\"segments\":[{\"start_secs\":0.0,\"end_secs\":60.0},{\"start_secs\":60.0,\"end_secs\":120.0}],\"output_format\":\"mp3\"}" \
  -o segments.zip
```

### Normalize volume
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/normalize \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@interview.wav" \
  -F "options={\"target_db\":-14.0,\"output_format\":\"wav\"}" \
  -o normalized.wav
```

### Waveform (returns PNG or JSON)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/waveform \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@song.mp3" \
  -F "options={\"output_format\":\"png\",\"width\":800,\"height\":200}" \
  -o waveform.png
```

### Metadata (returns JSON, no -o needed)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/audio/metadata \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "audio=@song.mp3"
```

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `output_format` | string | convert, merge, trim, split, normalize | - | "mp3", "wav", or "flac" |
| `bitrate` | integer | convert | - | MP3 kbps (128, 192, 320) |
| `sample_rate` | integer | convert | - | Hz (44100, 48000) |
| `channels` | integer | convert | - | 1 (mono) or 2 (stereo) |
| `start_secs` | float | trim | 0 | Start time in seconds |
| `end_secs` | float | trim | - | End time in seconds |
| `segments` | array | split | - | Array of `{start_secs, end_secs}` objects |
| `target_db` | float | normalize | -16.0 | Target dBFS |

## Formats

**Input:** MP3, WAV, FLAC, OGG, AAC, AIFF
**Output:** MP3, WAV, FLAC (OGG/AAC/AIFF input-only)

## After the call

Report: operation, input/output formats, file sizes, output path. Use `wc -c < file` for sizes.
