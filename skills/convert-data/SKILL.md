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

Convert between data formats via QkConvert. 25 conversion pairs. Costs 1 credit per request.

## Workflow

### 1. Parse the request

Identify the source file, its format (from extension or content), and the target format.

### 2. Build the endpoint path

Pattern: `/api/v1/data/{source}-to-{target}`

Examples:
- `/api/v1/data/csv-to-json`
- `/api/v1/data/json-to-toml`
- `/api/v1/data/yaml-to-xlsx`
- `/api/v1/data/md-to-html`

### 3. Call the API

Send the file as multipart with field name `file` or `data`. The response is the converted content (text for most formats, binary for XLSX).

### 4. Save and report

Save output with the appropriate extension. Show a preview of the first few lines for text formats.

## Conversion Matrix

Full bidirectional conversion between: **CSV, JSON, XML, YAML, XLSX, TOML** (20 pairs)
Plus: **TOML ↔ JSON**, **TOML ↔ YAML** (4 pairs)
Plus: **Markdown → HTML** (1 pair, one-way)

**Not supported:** TOML ↔ CSV/XLSX (hierarchical config vs flat tabular — incompatible structures)

## Parameters

| Parameter | Type | Applicable to | Default | Description |
|-----------|------|---------------|---------|-------------|
| `pretty` | boolean | *-to-json | false | Pretty-print JSON output |
| `delimiter` | string | csv-to-*, *-to-csv | "," | CSV delimiter. Also accepts "tab", ";", "\|" |
| `has_header` | boolean | csv-to-* | true | Whether first row is headers |
| `root_name` | string | *-to-xml | "root" | Root XML element name |
| `sheet` | string | xlsx-to-* | first sheet | Sheet name to read from XLSX |
| `full_document` | boolean | md-to-html | false | Wrap in full HTML document |
| `github_flavored` | boolean | md-to-html | true | Enable GFM tables, strikethrough, task lists |

## Format Notes

- **CSV** is untyped — numbers are inferred on parse, but strings and numbers may look different after round-trip
- **TOML** requires a top-level object (arrays at root will error), and has no null type (null values are silently stripped)
- **XLSX** is binary — only output as download, not previewable as text
- **XML** wraps arrays in a root element (configurable via `root_name`)
- **Markdown → HTML** is one-way only

## Errors

| Code | Meaning | User action |
|------|---------|-------------|
| 400 | Malformed input, unsupported conversion pair, non-UTF-8, or file > 10 MB | Check format and encoding |
| 401 | Invalid API key | Check QKCONVERT_API_KEY |
| 403 | Monthly credits exhausted | Upgrade plan or wait for reset |
| 429 | Rate limit exceeded | Wait for Retry-After seconds |

## Examples

Convert CSV to pretty-printed JSON:
```
/convert-data users.csv to json, pretty-printed
```

Convert a TOML config to YAML:
```
/convert-data config.toml to yaml
```

Convert an Excel spreadsheet to JSON:
```
/convert-data report.xlsx to json
```

Convert tab-separated data to XML with custom root:
```
/convert-data data.tsv to xml with tab delimiter and root element "records"
```

Render Markdown as full HTML document:
```
/convert-data readme.md to html as full document
```
