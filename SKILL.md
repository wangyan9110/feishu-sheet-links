# feishu-sheet-links

Extract all hyperlinks from a public Feishu spreadsheet across all sheet tabs.
Standalone skill — no dependency on other skills.

## Script Directory

Base: `.claude/skills/feishu-sheet-links/`

| File | Purpose |
|------|---------|
| `scripts/main.ts` | Step 1: Extract all hyperlinks from spreadsheet → JSON |
| `scripts/download.ts` | Step 2: Batch download linked articles → markdown files |
| `scripts/cdp.ts` | CDP utilities (Chrome launch, connection, evaluate) |

## Usage

### Step 1 — Extract links

```bash
SKILL_DIR="/path/to/.claude/skills/feishu-sheet-links"
npx -y bun ${SKILL_DIR}/scripts/main.ts <spreadsheet-url> [-o output.json]
```

**Example:**
```bash
npx -y bun ${SKILL_DIR}/scripts/main.ts \
  "https://xcngx0f1wik3.feishu.cn/wiki/XJexwR7Zqi2L80kcugtcFIfCnHg?sheet=ce2f5b" \
  -o feishu-links.json
```

### Step 2 — Batch download articles

```bash
npx -y bun ${SKILL_DIR}/scripts/download.ts <links.json> [-o output-dir] [-c concurrency] [--max-wait ms]
```

**Example:**
```bash
npx -y bun ${SKILL_DIR}/scripts/download.ts feishu-links.json \
  -o ./raw/shengcai \
  -c 5 \
  --max-wait 20000
```

**Options:**

| Option | Default | Description |
|--------|---------|-------------|
| `<links.json>` | — | Output from Step 1 |
| `-o <dir>` | `./feishu-articles` | Output directory for markdown files |
| `-c <n>` | `5` | Concurrent downloads |
| `--max-wait <ms>` | `20000` | Max page wait time per URL |

**Resume support**: Progress is saved to `<output-dir>/.download-progress.json` after each URL. Re-running the same command skips already-downloaded URLs.

## How It Works

1. **Reuses existing Chrome if available** — checks ports 64023, 9222, 9229 first.  
   Falls back to launching a new Chrome instance with its own profile (`feishu-sheet-links/chrome-profile`), isolated from other skills.
2. Opens the spreadsheet URL in a new tab.
3. Reads all sheet tab IDs from `spreadApp.collaborativeSpread._spread.sheetIdToIndexMap`.
4. Opens one tab per remaining sheet to trigger lazy data loading.
5. Extracts links from two Feishu storage formats:
   - **url-type** — `contentModel.link.idToRef._map` (whole-cell hyperlinks)
   - **mention-type** — `contentModel.segmentModel.table` (inline rich-text links)
6. Writes `{ "1月": [{text, url}, ...], ... }` JSON and prints a markdown summary.

## Output

```json
{
  "1月": [{ "text": "文章标题", "url": "https://my.feishu.cn/wiki/..." }],
  "2月": [...],
  "3月": [...],
  "4月": [...]
}
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `FEISHU_CHROME_PATH` | Custom Chrome executable |
| `FEISHU_CHROME_PROFILE` | Custom Chrome profile directory |

## Notes

- Works with **public/anonymous** Feishu spreadsheets (no login needed)
- 1月/2月/3月 typically use **url-type** links; 4月+ may use **mention-type**
- Each tab needs ~8–15 s to load; timeouts are set generously
- Chrome profile: `~/Library/Application Support/feishu-sheet-links/chrome-profile` (macOS)
