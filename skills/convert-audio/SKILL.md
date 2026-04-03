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

Process audio files via QkConvert.

## Workflow

### 1. Parse the request

Identify:
- **Source file(s)** — path to the audio file(s)
- **Operation** — convert, trim, merge, split, normalize, waveform, or metadata
- **Target format** — mp3, wav, flac (for conversion)
- **Options** — bitrate, start/end times, target dB

### 2. Choose the endpoint

| Need | Endpoint |
|------|----------|
| Change format | `/api/v1/audio/convert` |
| Cut to time range | `/api/v1/audio/trim` |
| Combine multiple files | `/api/v1/audio/merge` |
| Split by time or silence | `/api/v1/audio/split` |
| Adjust volume | `/api/v1/audio/normalize` |
| Visualize waveform | `/api/v1/audio/waveform` |
| Get duration/format info | `/api/v1/audio/metadata` |

### 3. Call the API

Send the file as multipart form data. For merge, send multiple files. Save the binary response to disk.

### 4. Report

Show: operation performed, input/output formats, duration, file sizes, output path.

## Supported Formats

Input: MP3, WAV, FLAC, OGG, AAC, AIFF
Output: MP3, WAV, FLAC

## Notes

- Each operation costs 3 credits
- Trim uses `start` and `end` in seconds (float)
- Normalize defaults to -16.0 dBFS
