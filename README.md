# AI 洞察日报

> 每日自动聚合 AI 领域动态，AI 生成中文摘要。基于 Cloudflare Workers 零成本运行。

## 🌐 在线阅读

**<https://ai-daily.352388732.workers.dev>**

- 最新一期：<https://ai-daily.352388732.workers.dev>
- 指定日期：<https://ai-daily.352388732.workers.dev/report?date=2026-10-13>
- RSS 订阅：<https://ai-daily.352388732.workers.dev/rss?days=7>

## 📡 数据源

| 类别 | 来源 | 说明 |
|---|---|---|
| 新闻 | Hacker News | Algolia 公开 API，按热度过滤 AI 相关话题 |
| 论文 | arXiv | cs.AI / cs.CL / cs.LG 最新投稿 |
| 项目 | GitHub Search API | 按 AI 相关 topic 检索近期高星仓库 |
| 社交 | Folo 列表 | 需登录 Cookie，未配置时自动跳过 |

## ⚙️ 工作流

每天 UTC 00:00（北京时间 08:00）自动执行：

```
抓取数据源 → 写入 KV → AI 生成日报 → 提交 GitHub → 生成 RSS → 推送 Telegram
```

| 组件 | 说明 |
|---|---|
| 运行环境 | Cloudflare Workers + KV |
| AI 模型 | Gemini Flash Lite（主）/ Groq gpt-oss-120b（自动降级） |
| 归档 | `daily/` 日报原文、`rss/` 精简版 |
| 通知 | Telegram Bot |

## 📁 目录结构

- `src/` — Worker 源码
- `src/dataSources/` — 各数据源抓取器
- `daily/` — 每日日报（Markdown）
- `rss/` — RSS 精简内容
- `docs/` — 部署文档

## 🚀 部署

参考 `wrangler.toml` 配置后执行 `wrangler deploy`。

## 📄 License

基于上游 [justlovemaki/CloudFlare-AI-Insight-Daily](https://github.com/justlovemaki/CloudFlare-AI-Insight-Daily) 修改，MIT。
