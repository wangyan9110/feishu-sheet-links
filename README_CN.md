English | [中文](README_CN.md)

# feishu-sheet-links

从公开的飞书多维表格中提取所有超链接——**所有 Sheet 标签页，不只是第一个**——并批量下载链接指向的文章为 Markdown 文件。

[![ClawHub](https://img.shields.io/badge/ClawHub-feishu--sheet--links-blue)](https://clawhub.ai/wangyan9110/feishu-sheet-links)
[![License: MIT-0](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)

## 适合谁用

用飞书多维表格管理文章链接的团队——文章库、内容日历、资料汇总——需要把这些链接（以及链接背后的内容）从飞书里取出来。

**典型场景：**
- **搭建本地知识库** — 把飞书表格里的几百篇文章下载为 Markdown，导入 LLM 或 RAG 管道
- **内容备份** — 文章消失或设为私密之前，批量存到本地
- **平台迁移** — 一次性提取所有链接和内容，不用逐篇手动打开
- **内容运营** — 团队在多 Sheet 的飞书表格里记录文章 URL，需要完整导出，不只是第一个标签页

**其他方案的问题：**
飞书侧边栏显示着所有 Sheet，但数据只有在你点击对应标签页时才会加载。抓 HTML、跑爬虫、调 API——你只能拿到第一个 Sheet，其余的根本不存在。

## 为什么不直接爬取？

飞书多维表格的每个 Sheet 只有在浏览器中被激活后才会加载数据模型。任何常规爬虫只能看到第一个 Sheet——其余的在 HTML 里根本不存在。

feishu-sheet-links 使用 **Chrome DevTools Protocol（CDP）** 逐一激活每个 Sheet，等待其内部 JavaScript 数据模型加载完成，再从飞书的两种链接格式（`url-type` 和 `mention-type`）中提取超链接。

**不需要 npm install。** 只要有 Bun 和 Chrome，直接运行。

## 环境要求

- [Bun](https://bun.sh) 运行时
- Google Chrome 或 Chromium

## 快速开始

```bash
# 第一步 — 提取所有 Sheet 的链接
npx -y bun scripts/main.ts "https://your-org.feishu.cn/wiki/..." -o links.json

# 第二步 — 批量下载为 Markdown
npx -y bun scripts/download.ts links.json -o ./articles
```

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

## 使用方法

### 第一步 — 提取链接

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

### 第二步 — 批量下载文章

```bash
npx -y bun scripts/download.ts <links.json> [-o output-dir] [-c concurrency] [--max-wait ms]
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-o <dir>` | `./feishu-articles` | 输出目录 |
| `-c <n>` | `5` | 并发下载数 |
| `--max-wait <ms>` | `20000` | 每个 URL 的最长等待时间 |

**并发下载：** 默认 5 个并发，几百篇文章几分钟搞定。

**断点续传：** 每成功下载一个 URL 后保存进度。随时可以中断，重新运行自动跳过已下载的文件。

## 工作原理

1. 优先复用已运行的 Chrome 实例（检测端口 64023、9222、9229），否则自动启动
2. 从 `spreadApp.collaborativeSpread._spread.sheetIdToIndexMap` 读取所有 Sheet ID
3. 对每个 Sheet 打开 `?sheet=<id>` 标签页，调用 `setActiveSheetIndex()` 触发懒加载，等待 `sheet._dataModel.contentModel` 填充
4. 从两种存储格式中提取链接：
   - **url-type：** `contentModel.link.idToRef._map`（整格超链接）
   - **mention-type：** `contentModel.segmentModel.table`（富文本内嵌链接）

## 环境变量

| 变量 | 说明 |
|------|------|
| `FEISHU_CHROME_PATH` | 自定义 Chrome 可执行文件路径 |
| `FEISHU_CHROME_PROFILE` | 自定义 Chrome Profile 目录 |

## 注意事项

- 仅支持**公开**飞书文档，无需登录
- 每个 Sheet 标签页加载约需 8–15 秒
- Chrome Profile 与系统默认 Profile 隔离：`~/Library/Application Support/feishu-sheet-links/chrome-profile`（macOS）

## License

[MIT-0](LICENSE) — 随意使用，无需署名。
