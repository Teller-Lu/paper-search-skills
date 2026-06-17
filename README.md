# Paper Search Skills for Claude Code

[English](#english) | [中文](#中文)

A unified Claude Code plugin that bundles 30 skills + 4 orchestrator agents for searching academic literature across **CNKI (中国知网)**, **Google Scholar**, **ScienceDirect (Elsevier)**, and **Web of Science** — all through Chrome DevTools MCP. Search, browse journals, extract metadata, download PDFs, and push to Zotero from a single Claude Code CLI.

---

<a id="english"></a>

## English

### Coverage

| Database | Prefix | Skills | Agent | Notes |
|---|---|---|---|---|
| [CNKI (中国知网)](https://www.cnki.net) | `cnki-` | 10 | `cnki-researcher` | Captcha-aware; supports SCI/EI/CSSCI/北大核心/CSCD source filtering |
| [Google Scholar](https://scholar.google.com) | `gs-` | 6 | `gs-researcher` | DOM-only (no public API); `data-cid` as primary key |
| [ScienceDirect (Elsevier)](https://www.sciencedirect.com) | `sd-` | 8 | `sd-researcher` | Institutional login recommended; Cloudflare-aware |
| [Web of Science](https://www.webofscience.com) | `wos-` | 6 | `wos-researcher` | API-first (`runQuerySearch`); JIF / JCR quartile extraction |

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Chrome browser
- [Zotero](https://www.zotero.org/) desktop app (optional, for citation export)
- Python 3 (optional, for the Zotero push script)
- Institutional login for ScienceDirect / Web of Science PDF download

### Installation

#### 1. Install Chrome DevTools MCP server

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. Install the plugin

```bash
git clone https://github.com/Teller-Lu/paper-search-skills.git
cd paper-search-skills
cp -r skills/ agents/ .claude/
```

Or add to an existing project:

```bash
git clone https://github.com/Teller-Lu/paper-search-skills.git /tmp/paper-search-skills
cp -r /tmp/paper-search-skills/skills/ your-project/.claude/skills/
cp -r /tmp/paper-search-skills/agents/ your-project/.claude/agents/
```

#### 3. Configure anti-detection (recommended for ScienceDirect / Web of Science)

Add to your `~/.claude.json` to avoid Cloudflare captcha loops and bot detection:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest",
        "--ignoreDefaultChromeArg=--enable-automation",
        "--ignoreDefaultChromeArg=--disable-infobars",
        "--chromeArg=--disable-blink-features=AutomationControlled"
      ]
    }
  }
}
```

#### 4. Start Chrome with remote debugging (only when not using the MCP-managed Chrome)

```bash
# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222
```

#### 5. Launch Claude Code

```bash
claude
```

Skills and agents are picked up automatically. Try `/cnki-search 深度学习` or `/gs-search deep learning` to verify.

### Skills

#### CNKI (10)

| Skill | Description | Invocation |
|---|---|---|
| `cnki-search` | Keyword search with structured result extraction | `/cnki-search 人工智能` |
| `cnki-advanced-search` | Filtered search: author, journal, date, source category (SCI/EI/CSSCI/北大核心/CSCD) | `/cnki-advanced-search CSSCI 人工智能 2020-2025` |
| `cnki-parse-results` | Re-parse an existing search results page | `/cnki-parse-results` |
| `cnki-navigate-pages` | Pagination and sort order | `/cnki-navigate-pages next` |
| `cnki-paper-detail` | Extract full paper metadata (abstract, keywords, etc.) | `/cnki-paper-detail` |
| `cnki-journal-search` | Find journals by name, ISSN, or CN | `/cnki-journal-search 计算机学报` |
| `cnki-journal-index` | Check journal indexing status and impact factors | `/cnki-journal-index 计算机学报` |
| `cnki-journal-toc` | Browse issue table of contents | `/cnki-journal-toc 计算机学报 2025 01期` |
| `cnki-download` | Download paper PDF/CAJ | `/cnki-download` |
| `cnki-export` | Push to Zotero or output GB/T 7714 citation | `/cnki-export zotero` |

#### Google Scholar (6)

| Skill | Description | Invocation |
|---|---|---|
| `gs-search` | Keyword search with structured result extraction | `/gs-search deep learning` |
| `gs-advanced-search` | Filtered search: author, journal, date range, exact phrase, title-only | `/gs-advanced-search author:Einstein after:2020 relativity` |
| `gs-cited-by` | Find papers that cite a given paper via `data-cid` | `/gs-cited-by 0qfs6zbVakoJ` |
| `gs-fulltext` | Get full-text access links: PDF, DOI, Sci-Hub, publisher | `/gs-fulltext 0qfs6zbVakoJ` |
| `gs-navigate-pages` | Pagination for search results | `/gs-navigate-pages next` |
| `gs-export` | Export to Zotero via BibTeX extraction | `/gs-export 0qfs6zbVakoJ` |

#### ScienceDirect (8)

| Skill | Description | Invocation |
|---|---|---|
| `sd-search` | Keyword search with structured result extraction | `/sd-search machine learning` |
| `sd-advanced-search` | Filtered search: author, journal, year, title, keywords | `/sd-advanced-search author: Smith year: 2024 deep learning` |
| `sd-parse-results` | Re-parse an existing search results page | _(internal)_ |
| `sd-navigate-pages` | Pagination, sort order, results-per-page | `/sd-navigate-pages next` |
| `sd-paper-detail` | Extract full article metadata (abstract, authors, DOI, etc.) | `/sd-paper-detail S0957417426005245` |
| `sd-journal-browse` | Browse journal info, impact factor, issues, articles | `/sd-journal-browse Nature Energy` |
| `sd-download` | Download article PDFs to local disk | `/sd-download S0957417426005245` |
| `sd-export` | Export citations as RIS/BibTeX/text, push to Zotero | `/sd-export S0957417426005245 format: zotero` |

#### Web of Science (6)

| Skill | Description | Invocation |
|---|---|---|
| `wos-search` | Search WoS by topic, author, title, DOI; supports edition/database filtering and sort | `/wos-search value co-creation --edition SSCI --sort citations` |
| `wos-paper-detail` | Extract full record metadata (abstract, JIF, keywords, citations, etc.) | `/wos-paper-detail WOS:000295471900004` |
| `wos-navigate-pages` | Paginate through search results via API or URL | `/wos-navigate-pages 2` |
| `wos-parse-results` | Parse search results from API response or DOM | _(internal)_ |
| `wos-download` | Download article PDFs via publisher links | `/wos-download WOS:000295471900004` |
| `wos-export` | Export citations to Zotero, RIS, BibTeX, or Excel | `/wos-export zotero` |

### Agents

| Agent | Orchestrates | Key Capabilities |
|---|---|---|
| `cnki-researcher` | All 10 CNKI skills | Captcha detection (pauses for manual solve), tab management, "search → filter → export" workflows |
| `gs-researcher` | All 6 Google Scholar skills | CAPTCHA detection, "search → cited-by → export" workflows |
| `sd-researcher` | All 8 ScienceDirect skills | Cloudflare captcha handling (auto-click or pause), "search → detail → export → download" workflows |
| `wos-researcher` | All 6 Web of Science skills | API-first with DOM fallback, Chinese keyword translation, "search → detail → export → download" workflows |

### How It Works

All skills use async `evaluate_script` calls through Chrome DevTools MCP — no screenshot parsing or OCR. Each skill operates in 1-2 tool calls (navigate + evaluate_script), making interactions fast and reliable.

Key design choices:

- **Single async script per operation** — replaces multi-step snapshot → click → wait_for patterns
- **Direct navigation over link clicking** — many of these sites open results in new tabs; navigating directly avoids tab management overhead
- **Batch export** — export multiple papers from a results page in one call instead of visiting each detail page
- **Captcha-aware** — detects Tencent slider (CNKI), Google reCAPTCHA, and Cloudflare challenges; pauses for manual resolution when needed
- **Primary key per source** — `data-cid` for Google Scholar; CNKI `dbcode/filename`; WoS `WOS:UT`; ScienceDirect `PII (S...)`
- **API-first where available** — Web of Science uses the internal `runQuerySearch` API for structured JSON; others fall back to DOM scraping
- **Zotero Connector API** — direct push via Connector API with deterministic session IDs to avoid duplicates

### Project Structure

```
paper-search-skills/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── cnki/          # 10 CNKI skills
│   ├── gs/            # 6 Google Scholar skills
│   ├── sd/            # 8 ScienceDirect skills
│   └── wos/           # 6 Web of Science skills
├── agents/
│   ├── cnki-researcher.md
│   ├── gs-researcher.md
│   ├── sd-researcher.md
│   └── wos-researcher.md
├── cnki_README.md     # original per-source READMEs
├── gs_README.md
├── sd_README.md
└── wos_README.md
```

---

<a id="中文"></a>

## 中文

一站式 Claude Code 插件，整合了 30 个 skill + 4 个调度 agent，覆盖 **CNKI 中国知网**、**Google Scholar 谷歌学术**、**ScienceDirect (Elsevier)**、**Web of Science** 四大学术数据库 —— 全部通过 Chrome DevTools MCP 完成。可在 Claude Code CLI 中完成检索、期刊浏览、元数据提取、PDF 下载、推送到 Zotero 等操作。

### 覆盖范围

| 数据库 | 前缀 | Skills | Agent | 说明 |
|---|---|---|---|---|
| [CNKI 中国知网](https://www.cnki.net) | `cnki-` | 10 | `cnki-researcher` | 支持验证码处理；支持 SCI/EI/CSSCI/北大核心/CSCD 来源筛选 |
| [Google Scholar 谷歌学术](https://scholar.google.com) | `gs-` | 6 | `gs-researcher` | 纯 DOM 解析（无公开 API）；以 `data-cid` 为主键 |
| [ScienceDirect (Elsevier)](https://www.sciencedirect.com) | `sd-` | 8 | `sd-researcher` | 建议机构账号登录；处理 Cloudflare |
| [Web of Science](https://www.webofscience.com) | `wos-` | 6 | `wos-researcher` | API 优先（`runQuerySearch`）；提取 JIF / JCR 分区 |

### 前置要求

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Chrome 浏览器
- [Zotero](https://www.zotero.org/) 桌面端（可选，用于引用导出）
- Python 3（可选，用于 Zotero 推送脚本）
- ScienceDirect / Web of Science PDF 下载需机构账号登录

### 安装步骤

#### 1. 安装 Chrome DevTools MCP 服务器

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. 安装本插件

```bash
git clone https://github.com/Teller-Lu/paper-search-skills.git
cd paper-search-skills
cp -r skills/ agents/ .claude/
```

或添加到已有项目：

```bash
git clone https://github.com/Teller-Lu/paper-search-skills.git /tmp/paper-search-skills
cp -r /tmp/paper-search-skills/skills/ your-project/.claude/skills/
cp -r /tmp/paper-search-skills/agents/ your-project/.claude/agents/
```

#### 3. 配置反检测（推荐，用于 ScienceDirect / Web of Science）

在 `~/.claude.json` 中添加：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest",
        "--ignoreDefaultChromeArg=--enable-automation",
        "--ignoreDefaultChromeArg=--disable-infobars",
        "--chromeArg=--disable-blink-features=AutomationControlled"
      ]
    }
  }
}
```

#### 4. 启动 Chrome 远程调试（仅在不使用 MCP 内置 Chrome 时）

```bash
# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222
```

#### 5. 启动 Claude Code

```bash
claude
```

技能和智能体会自动加载。输入 `/cnki-search 深度学习` 或 `/gs-search deep learning` 验证是否正常。

### 技能列表

#### CNKI（10 个）

| 技能 | 功能 | 调用方式 |
|---|---|---|
| `cnki-search` | 关键词检索，返回结构化结果 | `/cnki-search 人工智能` |
| `cnki-advanced-search` | 高级检索：作者、期刊、时间、来源类别（SCI/EI/CSSCI/北大核心/CSCD） | `/cnki-advanced-search CSSCI 人工智能 2020-2025` |
| `cnki-parse-results` | 重新解析当前搜索结果页 | `/cnki-parse-results` |
| `cnki-navigate-pages` | 翻页与排序 | `/cnki-navigate-pages next` |
| `cnki-paper-detail` | 提取论文详细信息（摘要、关键词等） | `/cnki-paper-detail` |
| `cnki-journal-search` | 按名称、ISSN、CN 号查找期刊 | `/cnki-journal-search 计算机学报` |
| `cnki-journal-index` | 查询期刊收录情况和影响因子 | `/cnki-journal-index 计算机学报` |
| `cnki-journal-toc` | 浏览期刊目录 | `/cnki-journal-toc 计算机学报 2025 01期` |
| `cnki-download` | 下载论文 PDF/CAJ | `/cnki-download` |
| `cnki-export` | 推送到 Zotero 或输出 GB/T 7714 引用 | `/cnki-export zotero` |

#### Google Scholar（6 个）

| 技能 | 功能 | 调用方式 |
|---|---|---|
| `gs-search` | 关键词搜索，返回结构化结果 | `/gs-search deep learning` |
| `gs-advanced-search` | 高级搜索：作者、期刊、时间、精确短语、仅标题 | `/gs-advanced-search author:Einstein after:2020 relativity` |
| `gs-cited-by` | 引用追踪：查找引用了某篇论文的所有文献 | `/gs-cited-by 0qfs6zbVakoJ` |
| `gs-fulltext` | 全文获取：PDF、DOI、Sci-Hub、出版商链接 | `/gs-fulltext 0qfs6zbVakoJ` |
| `gs-navigate-pages` | 搜索结果翻页 | `/gs-navigate-pages next` |
| `gs-export` | 通过 BibTeX 导出到 Zotero | `/gs-export 0qfs6zbVakoJ` |

#### ScienceDirect（8 个）

| 技能 | 功能 | 调用方式 |
|---|---|---|
| `sd-search` | 关键词检索，返回结构化结果 | `/sd-search machine learning` |
| `sd-advanced-search` | 高级检索：作者、期刊、年份、标题、关键词 | `/sd-advanced-search author: Smith year: 2024 deep learning` |
| `sd-parse-results` | 重新解析当前搜索结果页 | _（内部调用）_ |
| `sd-navigate-pages` | 翻页、排序、每页显示数量 | `/sd-navigate-pages next` |
| `sd-paper-detail` | 提取论文完整元数据（摘要、作者、DOI 等） | `/sd-paper-detail S0957417426005245` |
| `sd-journal-browse` | 浏览期刊信息、影响因子、期次、文章列表 | `/sd-journal-browse Nature Energy` |
| `sd-download` | 下载论文 PDF 到本地 | `/sd-download S0957417426005245` |
| `sd-export` | 导出引用为 RIS/BibTeX/纯文本，推送到 Zotero | `/sd-export S0957417426005245 format: zotero` |

#### Web of Science（6 个）

| 技能 | 功能 | 调用方式 |
|---|---|---|
| `wos-search` | 按主题、作者、标题、DOI 搜索，支持版本/数据库筛选和排序 | `/wos-search 价值共创 --edition SSCI --sort citations` |
| `wos-paper-detail` | 提取论文详细信息（摘要、JIF、关键词、引用数等） | `/wos-paper-detail WOS:000295471900004` |
| `wos-navigate-pages` | 通过 API 或 URL 翻页 | `/wos-navigate-pages 2` |
| `wos-parse-results` | 解析搜索结果（内部技能） | _（内部使用）_ |
| `wos-download` | 通过出版商链接下载 PDF | `/wos-download WOS:000295471900004` |
| `wos-export` | 导出引用到 Zotero、RIS、BibTeX 或 Excel | `/wos-export zotero` |

### 智能体

| Agent | 调度范围 | 关键能力 |
|---|---|---|
| `cnki-researcher` | 10 个 CNKI 技能 | 验证码检测（暂停等待手动完成）、标签页管理、"检索 → 筛选 → 导出"复合工作流 |
| `gs-researcher` | 6 个 Google Scholar 技能 | 验证码检测、"检索 → 引用追踪 → 导出"复合工作流 |
| `sd-researcher` | 8 个 ScienceDirect 技能 | Cloudflare 验证码处理（自动点击或暂停）、"检索 → 详情 → 导出 → 下载"复合工作流 |
| `wos-researcher` | 6 个 Web of Science 技能 | API 优先 + DOM 回退、中文关键词翻译、"检索 → 详情 → 导出 → 下载"复合工作流 |

### 工作原理

所有技能通过 Chrome DevTools MCP 的 `evaluate_script` 异步执行 JavaScript，无需截图识别或 OCR。每个技能仅需 1-2 次工具调用（导航 + 执行脚本），快速且稳定。

核心设计：

- **单次异步脚本** — 取代多步骤 snapshot → click → wait_for 模式
- **直接导航优于点击链接** — 许多站点会在新标签页打开链接，直接导航避免标签页管理开销
- **批量导出** — 从搜索结果页一次性导出多篇论文，无需逐篇进入详情页
- **验证码感知** — 检测腾讯滑块（CNKI）、Google reCAPTCHA、Cloudflare 等验证码并在需要时暂停等待手动处理
- **每个源都有主键** — Google Scholar 用 `data-cid`；CNKI 用 `dbcode/filename`；WoS 用 `WOS:UT`；ScienceDirect 用 `PII (S...)`
- **API 优先** — Web of Science 调用内部 `runQuerySearch` API 拿结构化 JSON；其余源退化到 DOM 解析
- **Zotero Connector API** — 通过 Connector API 直接推送，使用确定性会话 ID 防止重复

### 项目结构

```
paper-search-skills/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── cnki/          # 10 个 CNKI 技能
│   ├── gs/            # 6 个 Google Scholar 技能
│   ├── sd/            # 8 个 ScienceDirect 技能
│   └── wos/           # 6 个 Web of Science 技能
├── agents/
│   ├── cnki-researcher.md
│   ├── gs-researcher.md
│   ├── sd-researcher.md
│   └── wos-researcher.md
├── cnki_README.md     # 原始单源 README，保留作参考
├── gs_README.md
├── sd_README.md
└── wos_README.md
```

---

## License

MIT
