English | [中文](README_CN.md)

# feishu-sheet-links

**飞书多维表格链接提取 & 批量下载工具。** 不用再一个一个点标签页——一次提取所有 Sheet 里的全部链接，几分钟内批量下载为 Markdown 文件。

[![ClawHub](https://img.shields.io/badge/ClawHub-feishu--sheet--links-blue)](https://clawhub.ai/wangyan9110/feishu-sheet-links)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)

## 为什么飞书表格链接没法直接爬？

飞书多维表格的每个 Sheet 采用懒加载：数据只有在浏览器中点击对应标签页后才会加载。抓 HTML、跑爬虫、调飞书 API——你只能拿到第一个 Sheet，其余的根本不存在。

feishu-sheet-links 使用 **Chrome DevTools Protocol（CDP）** 逐一激活每个 Sheet，等待数据加载完成，再从飞书的两种链接格式（`url-type` 和 `mention-type`）中提取全部超链接。

**不需要 npm install。** 只要有 Bun 和 Chrome，直接运行。

## 适合谁用

你购买了一个付费知识库，内容主把精华文章的链接整理在飞书表格里。你想把这些文章全部下载到本地，做成自己的知识库。

更广泛地说：只要有人用飞书表格存链接——无论什么结构——这个工具就能帮你把所有 Sheet 里的链接全部提取并批量下载。

- **归档付费飞书知识库** — 在订阅到期或文章设为私密前，把内容全部存到本地
- **搭建本地 AI 知识库** — 批量下载为 Markdown，导入 LLM 或 RAG 管道
- **飞书文章批量导出** — 一次性提取全部链接，不用逐篇手动打开
- **备份飞书资料合集** — 任何用飞书表格整理文章链接的场景

## 环境要求

- [Bun](https://bun.sh) 运行时
- Google Chrome 或 Chromium

## 安装

**通过 ClawHub：**
```bash
clawhub install feishu-sheet-links
```

**通过 npx skills：**
```bash
npx skills add wangyan9110/feishu-sheet-links
```

**直接克隆：**
```bash
git clone https://github.com/wangyan9110/feishu-sheet-links
```

## 快速开始

```bash
# 第一步 — 提取飞书表格所有 Sheet 的链接
npx -y bun scripts/main.ts "https://your-org.feishu.cn/wiki/..." -o links.json

# 第二步 — 批量下载文章为 Markdown
npx -y bun scripts/download.ts links.json -o ./articles
```

第一步执行后的示例输出：

```
Found 4 sheets, 127 links total:
  1月: 32 links
  2月: 28 links
  3月: 35 links
  4月: 32 links

Saved to: links.json
```

## 使用方法

### 第一步 — 提取飞书表格链接

```bash
npx -y bun scripts/main.ts <spreadsheet-url> [-o output.json]
```

输出格式：
```json
{
  "1月": [{ "text": "文章标题", "url": "https://..." }],
  "2月": [...]
}
```

### 第二步 — 批量下载飞书文章

```bash
npx -y bun scripts/download.ts <links.json> [-o output-dir] [-c concurrency] [--max-wait ms]
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-o <dir>` | `./feishu-articles` | 输出目录 |
| `-c <n>` | `5` | 并发下载数 |
| `--max-wait <ms>` | `20000` | 每个 URL 最长等待时间 |

**默认 5 个并发**——几百篇文章几分钟搞定。

**断点续传：** 每成功下载一个 URL 后保存进度，随时可以中断，重新运行自动跳过已下载的文件。

## 环境变量

| 变量 | 说明 |
|------|------|
| `FEISHU_CHROME_PATH` | 自定义 Chrome 可执行文件路径 |
| `FEISHU_CHROME_PROFILE` | 自定义 Chrome Profile 目录 |

## 注意事项

- 仅支持**公开**飞书文档，无需登录
- 每个 Sheet 标签页加载约需 8–15 秒
- Chrome Profile 与系统默认 Profile 隔离：`~/Library/Application Support/feishu-sheet-links/chrome-profile`（macOS）

## 工作原理

1. 优先复用已运行的 Chrome 实例（检测端口 64023、9222、9229），否则自动启动
2. 从 `spreadApp.collaborativeSpread._spread.sheetIdToIndexMap` 读取所有 Sheet ID
3. 对每个 Sheet 打开 `?sheet=<id>` 标签页，调用 `setActiveSheetIndex()` 触发懒加载，等待 `sheet._dataModel.contentModel` 填充
4. 从两种存储格式中提取链接：
   - **url-type：** `contentModel.link.idToRef._map`（整格超链接）
   - **mention-type：** `contentModel.segmentModel.table`（富文本内嵌链接）

## License

[MIT-0](LICENSE) — 随意使用，无需署名。

---

**关键词：** 飞书多维表格提取链接 / 飞书表格批量下载 / 飞书知识库导出 Markdown / 飞书文章批量下载 / 飞书爬虫所有Sheet / 飞书懒加载解决方案 / 付费飞书知识库备份 / feishu scraper / feishu link extractor
