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

**Do not call the MCP tools directly** — the OpenAPI-to-MCP bridge cannot send multipart/form-data, so all tool calls arrive with empty content. Always use `curl` via the Bash tool.

The endpoint pattern is `/api/v1/data/{source}-to-{target}`:

**For text output (JSON, CSV, XML, YAML, TOML, HTML):**
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/{source}-to-{target} \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@{input_file}" \
  -F "options={json_options}"
```
Text responses print to stdout. Pipe to a file with `> output.json` or use `-o`.

**For binary output (XLSX):**
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/{source}-to-xlsx \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@{input_file}" \
  -o output.xlsx
```

**Rules:**
- Always use `-s` (silent)
- Use `-o` for binary (XLSX), redirect `>` or `-o` for text formats
- Field name is `file`
- Use escaped double quotes: `-F "options={\"pretty\":true}"`

## Curl commands — common conversions

### CSV to JSON (pretty)
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/csv-to-json \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@data.csv" \
  -F "options={\"pretty\":true}" \
  -o data.json
```

### JSON to CSV
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/json-to-csv \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@data.json" \
  -o data.csv
```

### JSON to YAML
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/json-to-yaml \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@config.json" \
  -o config.yaml
```

### YAML to TOML
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/yaml-to-toml \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@config.yaml" \
  -o config.toml
```

### TOML to JSON
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/toml-to-json \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@Cargo.toml" \
  -F "options={\"pretty\":true}" \
  -o cargo.json
```

### XLSX to JSON
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/xlsx-to-json \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@report.xlsx" \
  -F "options={\"pretty\":true}" \
  -o report.json
```

### Markdown to HTML
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/md-to-html \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@readme.md" \
  -F "options={\"full_document\":true,\"github_flavored\":true}" \
  -o readme.html
```

### CSV with custom delimiter
```bash
curl -s -X POST https://qkconvert.dev/api/v1/data/csv-to-json \
  -H "Authorization: Bearer $QKCONVERT_API_KEY" \
  -F "file=@data.tsv" \
  -F "options={\"delimiter\":\"tab\",\"pretty\":true}" \
  -o data.json
```

## All supported pairs

The endpoint is always `/api/v1/data/{from}-to-{to}`:

| From \ To | json | csv | xml | yaml | xlsx | toml | html |
|-----------|------|-----|-----|------|------|------|------|
| **csv**   | yes  | -   | yes | yes  | yes  | -    | -    |
| **json**  | -    | yes | yes | yes  | yes  | yes  | -    |
| **xml**   | yes  | yes | -   | yes  | yes  | -    | -    |
| **yaml**  | yes  | yes | yes | -    | yes  | yes  | -    |
| **xlsx**  | yes  | yes | yes | yes  | -    | -    | -    |
| **toml**  | yes  | -   | -   | yes  | -    | -    | -    |
| **md**    | -    | -   | -   | -    | -    | -    | yes  |

## Parameters

| Parameter | Type | For | Default | Description |
|-----------|------|-----|---------|-------------|
| `pretty` | boolean | *-to-json | false | Pretty-print JSON |
| `delimiter` | string | csv-to-*, *-to-csv | "," | "tab", ";", "\|" |
| `has_header` | boolean | csv-to-* | true | First row is headers |
| `root_name` | string | *-to-xml | "root" | Root XML element name |
| `sheet` | string | xlsx-to-* | first | Sheet name to read |
| `full_document` | boolean | md-to-html | false | Full HTML wrapper |
| `github_flavored` | boolean | md-to-html | true | GFM tables, strikethrough, task lists |

## Format notes

- **TOML**: requires top-level object (no arrays at root), no null values (silently stripped)
- **CSV**: untyped — numbers inferred on parse
- **XLSX**: binary format — always use `-o` for output
- **Markdown to HTML**: one-way only

## After the call

For text formats, show a preview of the first few lines. Report file sizes with `wc -c < file`.
