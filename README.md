[中文](README_CN.md) | English

# feishu-sheet-links

Extract all hyperlinks from a public Feishu spreadsheet — across **all sheet tabs** — and optionally batch-download the linked articles as Markdown files.

[![ClawHub](https://img.shields.io/badge/ClawHub-feishu--sheet--links-blue)](https://clawhub.ai/wangyan9110/feishu-sheet-links)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)

## Why This Exists

Feishu spreadsheets lazy-load each sheet's data model only when that tab is activated in the browser. Standard scraping misses every sheet except the first one.

This tool uses **Chrome DevTools Protocol (CDP)** to activate each sheet programmatically, wait for its internal JavaScript model to populate, then extract links from both Feishu storage formats (`url-type` and `mention-type`).

**Zero npm dependencies.** Requires only Bun and Chrome.

## Requirements

- [Bun](https://bun.sh) runtime
- Google Chrome or Chromium

## Quick Start

```bash
npx -y bun scripts/main.ts "https://your-org.feishu.cn/wiki/..." -o links.json
npx -y bun scripts/download.ts links.json -o ./articles
```

## Installation

**As a Claude Code skill — via ClawHub:**
```bash
clawhub install feishu-sheet-links
```

**As a Claude Code skill — via npx skills:**
```bash
npx skills add wangyan9110/feishu-sheet-links
```

**Clone and run directly:**
```bash
git clone https://github.com/wangyan9110/feishu-sheet-links
cd feishu-sheet-links
npx -y bun scripts/main.ts "<url>"
```

## Usage

### Extract links

```bash
npx -y bun scripts/main.ts <spreadsheet-url> [-o output.json]
```

Output:
```json
{
  "Sheet1": [{ "text": "Article Title", "url": "https://..." }],
  "Sheet2": [...]
}
```

### Batch download articles

```bash
npx -y bun scripts/download.ts <links.json> [-o output-dir] [-c concurrency] [--max-wait ms]
```

| Option | Default | Description |
|--------|---------|-------------|
| `-o <dir>` | `./feishu-articles` | Output directory |
| `-c <n>` | `5` | Concurrent downloads |
| `--max-wait <ms>` | `20000` | Per-URL timeout |

Resume support: progress is saved after each URL, re-running skips already-downloaded files.

### Use in Claude Code

After installing as a skill, invoke:

```
/feishu-sheet-links https://your-org.feishu.cn/wiki/...
```

Claude will extract links, show a summary, then offer to batch-download as Markdown.

## How It Works

1. Connects to an existing Chrome instance (ports 64023, 9222, 9229) or launches its own
2. Reads all sheet IDs from `spreadApp.collaborativeSpread._spread.sheetIdToIndexMap`
3. For each sheet, opens a tab at `?sheet=<id>`, calls `setActiveSheetIndex()` to trigger lazy loading, waits for `sheet._dataModel.contentModel`
4. Extracts links from both storage formats:
   - **url-type:** `contentModel.link.idToRef._map` — whole-cell hyperlinks
   - **mention-type:** `contentModel.segmentModel.table` — inline rich-text links

## Environment Variables

| Variable | Description |
|----------|-------------|
| `FEISHU_CHROME_PATH` | Custom Chrome executable path |
| `FEISHU_CHROME_PROFILE` | Custom Chrome profile directory |

## Notes

- Works with **public** Feishu documents only (no login required)
- Each sheet tab takes 8–15 seconds to load
- Chrome profile is isolated from your system profile: `~/Library/Application Support/feishu-sheet-links/chrome-profile` (macOS)

## License

[MIT-0](LICENSE) — use freely, no attribution required.
