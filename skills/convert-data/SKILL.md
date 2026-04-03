---
name: convert-data
description: >
  Convert between data formats using the QkConvert API. Use when the user asks to
  "convert CSV to JSON", "convert JSON to YAML", "convert XML to CSV",
  "convert TOML to JSON", "convert XLSX to JSON", "convert Markdown to HTML",
  or needs to transform between CSV, JSON, XML, YAML, XLSX, TOML, and Markdown.
argument-hint: "<file> to <format>"
---

# /convert-data

Convert between data formats via QkConvert. 25 conversion pairs supported.

## Workflow

### 1. Parse the request

Identify:
- **Source file** — path to the data file
- **Source format** — csv, json, xml, yaml, xlsx, toml, md
- **Target format** — csv, json, xml, yaml, xlsx, toml, html

### 2. Build the endpoint path

The endpoint follows the pattern: `/api/v1/data/{source}-to-{target}`

Examples: `/api/v1/data/csv-to-json`, `/api/v1/data/json-to-toml`, `/api/v1/data/md-to-html`

### 3. Call the API

Send the file as multipart form data. The response is the converted text.

### 4. Save and report

Save the output to a file (same name, new extension). Show a preview of the first few lines.

## Conversion Matrix

All bidirectional pairs between CSV, JSON, XML, YAML, XLSX, and TOML are supported (20 pairs). Plus TOML to/from JSON and YAML (4 pairs). Plus Markdown to HTML (1 pair).

## Options

- `pretty` (boolean) — pretty-print JSON output (csv-to-json, xml-to-json, yaml-to-json, toml-to-json)
- `delimiter` (string) — CSV delimiter: comma (default), "tab", semicolon, pipe
- `has_header` (boolean) — whether CSV has a header row (default: true)
- `root_name` (string) — root XML element name (default: "root")

## Notes

- TOML requires a top-level object (arrays at root will error)
- TOML has no null type (null values are stripped silently)
- Each operation costs 1 credit
