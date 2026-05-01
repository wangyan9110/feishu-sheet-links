[中文](README_CN.md) | English

# feishu-sheet-links

Stop opening Feishu tabs one by one. Extract every link from every sheet — then download all the articles as Markdown in minutes.

[![ClawHub](https://img.shields.io/badge/ClawHub-feishu--sheet--links-blue)](https://clawhub.ai/wangyan9110/feishu-sheet-links)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)

## The Problem

Feishu spreadsheets show all your sheets in the sidebar, but each sheet's data only loads when you click the tab. Copy the HTML, run a scraper, call the API — you get one sheet. The rest are invisible.

feishu-sheet-links uses **Chrome DevTools Protocol (CDP)** to activate every sheet programmatically, wait for the data to load, and extract links from both Feishu storage formats (`url-type` and `mention-type`).

**No npm install.** Just Bun and Chrome.

## Who Is This For

You paid for a knowledge subscription. The curator shared hundreds of article links in a Feishu spreadsheet. You want it all on your local machine.

More broadly: if someone is using a Feishu spreadsheet to store links — any links, any structure — this tool downloads them all, across every sheet tab.

- **Archive a paid knowledge base** — download everything before your subscription expires or articles go private
- **Build a local knowledge base** — pull articles as Markdown, feed to an LLM or RAG pipeline
- **Migrate content** — extract everything in one pass instead of opening each link manually
- **Back up research collections** — any Feishu spreadsheet used as a link library

## Requirements

- [Bun](https://bun.sh) runtime
- Google Chrome or Chromium

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

## Quick Start

```bash
# Step 1 — extract links from all sheets
npx -y bun scripts/main.ts "https://your-org.feishu.cn/wiki/..." -o links.json

# Step 2 — batch download as Markdown
npx -y bun scripts/download.ts links.json -o ./articles
```

Example output after Step 1:

```
Found 4 sheets, 127 links total:
  1月: 32 links
  2月: 28 links
  3月: 35 links
  4月: 32 links

Saved to: links.json
```

## Usage

### Step 1 — Extract links

```bash
npx -y bun scripts/main.ts <spreadsheet-url> [-o output.json]
```

Output format:
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

**5 parallel workers by default** — hundreds of articles done in minutes.

**Resume support:** progress is saved after each download. Kill it anytime — re-running skips already-downloaded files.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `FEISHU_CHROME_PATH` | Custom Chrome executable path |
| `FEISHU_CHROME_PROFILE` | Custom Chrome profile directory |

## Notes

- Works with **public** Feishu documents only (no login required)
- Each sheet tab takes 8–15 seconds to load
- Chrome profile is isolated from your system profile: `~/Library/Application Support/feishu-sheet-links/chrome-profile` (macOS)

## How It Works

1. Connects to an existing Chrome instance (ports 64023, 9222, 9229) or launches its own
2. Reads all sheet IDs from `spreadApp.collaborativeSpread._spread.sheetIdToIndexMap`
3. For each sheet, opens a tab at `?sheet=<id>`, calls `setActiveSheetIndex()` to trigger lazy loading, waits for `sheet._dataModel.contentModel`
4. Extracts links from both storage formats:
   - **url-type:** `contentModel.link.idToRef._map` — whole-cell hyperlinks
   - **mention-type:** `contentModel.segmentModel.table` — inline rich-text links

## License

[MIT-0](LICENSE) — use freely, no attribution required.
