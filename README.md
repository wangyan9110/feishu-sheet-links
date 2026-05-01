[中文](README_CN.md) | English

# feishu-sheet-links

> A Claude Code skill that extracts all hyperlinks from a public Feishu spreadsheet — across **all sheet tabs** — and optionally batch-downloads the linked articles as Markdown files.

[![ClawHub](https://img.shields.io/badge/ClawHub-feishu--sheet--links-blue)](https://clawhub.ai/wangyan9110/feishu-sheet-links)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)

## Why This Exists

Feishu spreadsheets lazy-load each sheet's data model only when that tab is activated in the browser. Standard scraping approaches miss every sheet except the first one.

This tool uses **Chrome DevTools Protocol (CDP)** to programmatically activate each sheet, wait for its internal JavaScript model to populate, then extract links from two different Feishu storage formats (`url-type` and `mention-type`).

**Zero npm dependencies.** No `npm install` needed — only Bun and Chrome.

## Requirements

- [Bun](https://bun.sh) runtime
- Google Chrome (or Chromium)

## Installation

**Via ClawHub** (recommended):
```bash
clawhub install feishu-sheet-links
```

**Via npx skills:**
```bash
npx skills add wangyan9110/feishu-sheet-links
```

**Manual:**
```bash
git clone https://github.com/wangyan9110/feishu-sheet-links
cp -r feishu-sheet-links ~/.claude/skills/
```

## Usage in Claude Code

Once installed, invoke the skill in Claude Code:

```
/feishu-sheet-links https://your-org.feishu.cn/wiki/...
```

Claude will:
1. Extract all hyperlinks from every sheet tab
2. Show a summary (sheet names + link counts)
3. Offer to batch-download the linked articles as Markdown

## Use as Standalone Scripts

**Step 1 — Extract links from all sheets:**

```bash
npx -y bun scripts/main.ts "https://your-org.feishu.cn/wiki/..." -o links.json
```

Output format:
```json
{
  "Sheet1": [{ "text": "Article Title", "url": "https://..." }],
  "Sheet2": [...]
}
```

**Step 2 — Batch download the linked articles as Markdown:**

```bash
npx -y bun scripts/download.ts links.json -o ./articles -c 5
```

| Option | Default | Description |
|--------|---------|-------------|
| `-o <dir>` | `./feishu-articles` | Output directory |
| `-c <n>` | `5` | Concurrent downloads |
| `--max-wait <ms>` | `20000` | Per-URL timeout |

**Resume support:** Progress is saved after each URL. Re-running skips already-downloaded files.

## How It Works

1. Connects to an existing Chrome instance if one is running (ports 64023, 9222, 9229), otherwise launches its own
2. Opens the spreadsheet to discover all sheet IDs from `spreadApp.collaborativeSpread._spread.sheetIdToIndexMap`
3. For each sheet, opens a dedicated tab at `?sheet=<id>`, calls `setActiveSheetIndex()` to trigger lazy loading, and waits for `sheet._dataModel.contentModel` to populate
4. Extracts links from both storage formats:
   - **url-type:** `contentModel.link.idToRef._map` (whole-cell hyperlinks)
   - **mention-type:** `contentModel.segmentModel.table` (inline rich-text links)

## Environment Variables

| Variable | Description |
|----------|-------------|
| `FEISHU_CHROME_PATH` | Custom Chrome executable path |
| `FEISHU_CHROME_PROFILE` | Custom Chrome profile directory |

## Notes

- Works with **public** Feishu documents (no login required)
- Each sheet tab takes 8–15 seconds to load; timeouts are set generously
- Chrome profile is isolated from your default profile: `~/Library/Application Support/feishu-sheet-links/chrome-profile` (macOS)

## License

[MIT-0](LICENSE) — No attribution required.
