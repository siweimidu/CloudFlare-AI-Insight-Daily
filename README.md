# AI 洞察日报

> 每日自动聚合 AI 动态、开源项目与论文，AI 生成中文摘要。

## 🌐 在线阅读

**<https://ai-daily.352388732.workers.dev>**

- 最新一期：<https://ai-daily.352388732.workers.dev> 或 <https://ai-daily.352388732.workers.dev/report>
- 指定日期：<https://ai-daily.352388732.workers.dev/report?date=2026-10-13>
- RSS 订阅：<https://ai-daily.352388732.workers.dev/rss?days=7>

## ⚙️ 技术架构

基于 **Cloudflare Workers + KV**，每天 UTC 00:00（北京时间 08:00）自动执行：

```
抓取数据源 → 写入 KV → Gemini/Groq 生成日报 → 提交 GitHub → 生成 RSS → 推送 Telegram
```

| 组件 | 说明 |
|---|---|
| 数据源 | GitHub Search API（近 7 天热门新建仓库） |
| AI 模型 | Gemini Flash Lite（主）/ Groq gpt-oss-120b（备） |
| 存储 | Cloudflare KV |
| 归档 | 本仓库 `daily/` 与 `rss/` 目录 |
| 推送 | Telegram Bot |

## 📁 目录结构

- `src/` — Worker 源码
- `daily/` — 每日日报原文（Markdown）
- `rss/` — RSS 精简版内容
- `docs/` — 部署文档

## 🚀 部署

见 [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)，或直接参考 `wrangler.toml` 配置后用 `wrangler deploy` 部署。

## 📄 License

本仓库基于上游 [justlovemaki/CloudFlare-AI-Insight-Daily](https://github.com/justlovemaki/CloudFlare-AI-Insight-Daily) 修改。

MIT
