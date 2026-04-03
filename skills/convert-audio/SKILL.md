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

## Workflow

### 1. Parse the request

Identify the source file(s), operation, target format, and options. For merge, collect all input files.

### 2. Choose the endpoint

| Need | Endpoint | Key parameters |
|------|----------|----------------|
| Change format | `/api/v1/audio/convert` | `output_format` (required) |
| Cut to time range | `/api/v1/audio/trim` | `start`, `end` in seconds |
| Combine files | `/api/v1/audio/merge` | Multiple files, `output_format` |
| Split by time/silence | `/api/v1/audio/split` | Split parameters |
| Adjust volume | `/api/v1/audio/normalize` | `target_db` |
| Visualize waveform | `/api/v1/audio/waveform` | `width`, `height`, `format` |
| Get duration/info | `/api/v1/audio/metadata` | None (returns JSON) |

### 3. Call the API

Send with field name `audio` or `file`. For merge, send multiple `audio` fields. Save the binary response to disk.

### 4. Report

Tell the user: operation, input/output formats, duration if available, file sizes, output path.

## Parameters

| Parameter | Type | Endpoints | Default | Description |
|-----------|------|-----------|---------|-------------|
| `output_format` | string | convert, merge | - | Required. "mp3", "wav", or "flac" |
| `bitrate` | integer | convert | - | MP3 bitrate in kbps (e.g. 128, 192, 320) |
| `sample_rate` | integer | convert | - | Sample rate in Hz (e.g. 44100, 48000) |
| `channels` | integer | convert | - | 1 (mono) or 2 (stereo) |
| `start` | float | trim | 0 | Start time in seconds |
| `end` | float | trim | - | End time in seconds |
| `target_db` | float | normalize | -16.0 | Target loudness in dBFS |

## Formats

**Input:** MP3, WAV, FLAC, OGG, AAC, AIFF
**Output:** MP3, WAV, FLAC

OGG, AAC, and AIFF can be decoded (input) but not encoded (output).

## Errors

| Code | Meaning | User action |
|------|---------|-------------|
| 400 | Unsupported format, corrupt file, invalid time range, or file > 10 MB | Check format and parameters |
| 401 | Invalid API key | Check QKCONVERT_API_KEY |
| 403 | Monthly credits exhausted | Upgrade plan or wait for reset |
| 429 | Rate limit exceeded | Wait for Retry-After seconds |

## Examples

Convert WAV to MP3 at 192kbps:
```
/convert-audio recording.wav to mp3 at 192kbps
```

Trim a podcast clip:
```
/convert-audio episode.mp3 trim from 1:30 to 5:45
```

Merge multiple audio files:
```
/convert-audio merge intro.mp3 main.mp3 outro.mp3 as flac
```

Normalize volume:
```
/convert-audio normalize interview.wav to -14 dBFS
```
