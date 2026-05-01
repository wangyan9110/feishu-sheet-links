[中文](README_CN.md) | English

# feishu-sheet-links

Extract all hyperlinks from a public Feishu spreadsheet — **every sheet tab, not just the first** — and batch-download the linked articles as Markdown files.

[![ClawHub](https://img.shields.io/badge/ClawHub-feishu--sheet--links-blue)](https://clawhub.ai/wangyan9110/feishu-sheet-links)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)

## Why Not Just Scrape It Yourself?

Feishu spreadsheets lazy-load each sheet's data model only when that tab is activated in a real browser. Any standard scraper only sees the first sheet — the rest simply don't exist in the HTML.

feishu-sheet-links uses **Chrome DevTools Protocol (CDP)** to activate each sheet programmatically and wait for its internal JavaScript model to populate, then extracts links from both Feishu storage formats (`url-type` and `mention-type`).

**No npm install.** Drop in Bun and Chrome and you're done.

## Requirements

- [Bun](https://bun.sh) runtime
- Google Chrome or Chromium

## Quick Start

```bash
# Step 1 — extract links from all sheets
npx -y bun scripts/main.ts "https://your-org.feishu.cn/wiki/..." -o links.json

# Step 2 — batch download as Markdown
npx -y bun scripts/download.ts links.json -o ./articles
```

## Installation

**Via ClawHub:**
```bash
clawhub install feishu-sheet-links
```

**Via npx skills:**
```bash
npx skills add wangyan9110/feishu-sheet-links
```

**Clone directly:**
```bash
git clone https://github.com/wangyan9110/feishu-sheet-links
```

## Usage

### Step 1 — Extract links

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

### Step 2 — Batch download articles

```bash
npx -y bun scripts/download.ts <links.json> [-o output-dir] [-c concurrency] [--max-wait ms]
```

| Option | Default | Description |
|--------|---------|-------------|
| `-o <dir>` | `./feishu-articles` | Output directory |
| `-c <n>` | `5` | Concurrent downloads |
| `--max-wait <ms>` | `20000` | Per-URL timeout |

**Resume support:** progress is saved after each successful download. Kill it anytime — re-running picks up where it left off.

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
