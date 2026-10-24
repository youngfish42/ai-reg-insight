# 使用与部署文档

本文档记录 **AI 法规瞭望 (AI Reg Insight)** 的运行方式、参数说明与部署细节。项目门户与概览见 [README](../README.md)。

## 目标

使用检索词 **“智能”**（可通过参数修改），在标题检索后分别查询：

1. **法律法规 → 中央法规**：筛选 **“本月生效”**
2. **法律法规 → 地方法规**：筛选 **“本月生效”**
3. **法律法规 → 立法资料**：筛选 **“本月生效”**
4. **法律法规 → 法律动态**：筛选 **“本月生效”**

并导出相应结果的：**title / url / publish_date** 等字段。

## 环境准备

- Python 3.10+（建议 3.12）
- 安装依赖：

```bash
pip install -r requirements.txt
playwright install chromium
```

## 快速运行

```bash
python query.py --keyword 智能 --out 法规.csv
```

同时输出 JSON：

```bash
python query.py --keyword 智能 --out 法规.csv --out-json results.json
```

## 常用参数

- `--keyword`：检索词（默认：智能）
- `--filter-keywords`：标题二次过滤关键词，逗号分隔（默认使用 `--keyword` 的值）；仅标题中包含至少一个关键词的条目才会被保存
- `--headless / --headed`：无头/有头模式（默认无头）
- `--max-items`：最多查询多少条（0 表示不限制，默认 0；对四个分类都生效）
- `--slow-mo`：调试用，放慢浏览器操作（毫秒）
- `--user-data-dir`：使用持久化浏览器目录保存 cookie（可用于需要登录的情况）
- `--source mcp`：通过北大法宝 MCP 服务检索（需设置 `PKULAW_MCP_TOKEN` 环境变量）
- `--month YYYY.MM`：回填历史月份，用于补录遗漏数据
- `--enrich-existing`：仅补全已有条目的制定机关 / 效力位阶

示例（有头 + 放慢 + 限制 200 条）：

```bash
python query.py --headed --slow-mo 200 --max-items 200
```

更多命令示例（MCP 检索、历史回填、RSS 生成等）见 [AGENTS.md](../AGENTS.md) 第 5 节。

## 登录/权限说明

- 本脚本只查询**列表页**信息，通常无需登录。
- 若你在本地运行时遇到“需要登录/验证码/无法翻页”等情况：
  1. 用 `--headed --user-data-dir ./user_data` 运行一次
  2. 在弹出的浏览器里手动登录
  3. 以后继续用相同的 `--user-data-dir` 运行即可复用登录态

## 输出格式

`法规.csv` 列：

| 列名 | 含义 |
| --- | --- |
| `category` | 中央法规 / 地方法规 / 立法资料 / 法规解读 / 法律动态 |
| `title` | 法规标题（去重键） |
| `url` | 详情页链接 |
| `publish_date` | 公布日期，YYYY.MM.DD 或 YYYY.MM |
| `issuing_authority` | 制定机关 |
| `legal_hierarchy` | 效力位阶 |
| `effective_date` | 施行日期 |
| `source` | 数据来源（browser / mcp），仅用于落盘存储 |
| `timeliness` | 时效性（仅 MCP 渠道有） |
| `document_no` | 发文字号（仅 MCP 渠道有） |
| `subject_tags` | 主题分类标签（仅 MCP 渠道有） |

## GitHub Pages 部署

本项目包含自动更新 GitHub Pages 的功能。

1. **启用 GitHub Pages**：
   - 进入仓库 **Settings** -> **Pages**。
   - 在 **Source** 下选择 **Deploy from a branch**。
   - 在 **Branch** 下选择 `main` 分支（或你当前的分支），文件夹选择 `/ (root)`。
   - 点击 **Save**。

2. **自动更新**：
   - GitHub Actions 每天定时运行检索任务（MCP 优先，失败时回退浏览器途径）。
   - 若有新法规，`法规.csv` 会被更新，网页与 `feed.xml` 也会随之更新。
   - 你也可以在 **Actions** 标签页手动运行 "Update Law Query Data" 工作流。

3. **访问网页**：
   - 设置完成后，网页地址通常为 `https://<你的用户名>.github.io/ai-reg-insight/`。

## 免责声明

仅用于学习研究与合规的信息获取，请遵守网站服务条款与相关法律法规。
