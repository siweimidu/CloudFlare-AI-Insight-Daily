# AI 洞察日报 · 后端

> 每天自动抓取 AI 领域动态，用大模型生成中文日报，并自动归档到 GitHub、输出 RSS、推送到 Telegram。
> 基于 **Cloudflare Workers + KV** 运行，**零服务器、零成本**。

<p>
  <img alt="Cloudflare Workers" src="https://img.shields.io/badge/Cloudflare-Workers-orange">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-blue">
  <img alt="Cost" src="https://img.shields.io/badge/cost-%E5%85%8D%E8%B4%B9-brightgreen">
</p>

---

## 📖 这是什么

一个跑在 Cloudflare 边缘网络上的定时任务，每天北京时间早上 8 点自动完成：

```
抓取数据源 → 存入 KV → 大模型生成中文日报 → 提交到 GitHub → 生成 RSS → 推送 Telegram
```

**你不需要有服务器。** 整个系统跑在 Cloudflare 免费额度内。

---

## 🌐 在线地址

| 用途 | 地址 |
|---|---|
| **日报展示站（推荐）** | **https://siweimidu.github.io/ai-daily-site/** |
| 后端 API 入口 | https://ai-daily.352388732.workers.dev |
| 最新一期 | https://ai-daily.352388732.workers.dev/report |
| 指定日期 | https://ai-daily.352388732.workers.dev/report?date=2026-10-13 |
| RSS 订阅 | https://ai-daily.352388732.workers.dev/rss?days=7 |

> 展示站是独立仓库 [ai-daily-site](https://github.com/siweimidu/ai-daily-site)（Hugo + Hextra），
> 每天自动从本仓库的 `daily/` 目录同步内容。

---

## 📡 数据源

| 类别 | 来源 | 是否需要 Key | 说明 |
|---|---|---|---|
| **新闻** | Hacker News | 免费无需 | 通过 Algolia 公开 API，按热度 + AI 关键词双重过滤 |
| **论文** | arXiv | 免费无需 | cs.AI / cs.CL / cs.LG 三个分类的最新投稿 |
| **项目** | GitHub Search API | 建议配置 | 按 AI 相关 topic 检索近期高星仓库 |
| **社交** | Folo | 需要 Cookie | 可选，未配置时自动跳过，不影响其他源 |

**过滤规则：**

- **Hacker News**：只看最近 2 天、热度 > 50 分，且标题命中 AI 相关关键词（ai / llm / gpt / agent / model 等）的帖子
- **GitHub**：从 `ai,llm,agents,machine-learning,generative-ai` 这几个 topic 分别检索后合并去重，避免混入无关项目
- **Folo**：需要登录态 Cookie。不配置时这几个源会静默跳过

---

## 🚀 部署教程

### 前置准备

| 需要什么 | 说明 |
|---|---|
| Cloudflare 账号 | 免费注册，Workers 免费额度每天 10 万次请求 |
| Node.js 18+ | 用于运行 wrangler 部署工具 |
| GitHub Token | 用于把日报提交回仓库，[创建地址](https://github.com/settings/tokens)（勾选 `repo` 权限） |
| 大模型 API Key | 二选一即可，见下方 |

**大模型选择：**

| 提供商 | 免费额度 | 获取地址 |
|---|---|---|
| Google Gemini | 每天 500 次 | https://aistudio.google.com/apikey |
| Groq | 每天 20 万 token | https://console.groq.com/keys |

> 建议用 Gemini 作主模型、Groq 作备用。代码内置自动降级：主模型失败时自动切到备用。

---

### 第 1 步：克隆代码

```bash
git clone https://github.com/siweimidu/CloudFlare-AI-Insight-Daily.git
cd CloudFlare-AI-Insight-Daily
```

### 第 2 步：安装部署工具

```bash
npm install -g wrangler
wrangler login          # 会打开浏览器授权
```

### 第 3 步：创建 KV 存储

```bash
wrangler kv namespace create DATA_KV
```

命令会输出类似这样的内容，**记下 `id` 的值**：

```
[[kv_namespaces]]
binding = "DATA_KV"
id = "a510e522dd5a4b7eb101564e7b749198"
```

### 第 4 步：填写配置

打开 `wrangler.toml`，把第 3 步拿到的 `id` 填进去，然后按注释修改下面的项：

```toml
kv_namespaces = [
  { binding = "DATA_KV", id = "这里填你的 KV ID" }
]

[vars]
# ========== 必填 ==========
GEMINI_API_KEY = "你的 Gemini Key"
GITHUB_TOKEN = "你的 GitHub Token"
GITHUB_REPO_OWNER = "你的 GitHub 用户名"
GITHUB_REPO_NAME = "CloudFlare-AI-Insight-Daily"
GITHUB_BRANCH = "main"
LOGIN_USERNAME = "admin"           # 管理后台账号
LOGIN_PASSWORD = "改成你自己的密码"  # 管理后台密码
CRON_TOKEN = "随便一串随机字符"      # 手动触发定时任务的令牌

# ========== 备用模型（可选）==========
USE_MODEL_PLATFORM = "GEMINI"      # 主模型平台，GEMINI 或 OPEN
OPENAI_API_KEY = "你的 Groq Key"
OPENAI_API_URL = "https://api.groq.com/openai"
DEFAULT_OPEN_MODEL = "openai/gpt-oss-120b"

# ========== Telegram 推送（可选）==========
TELEGRAM_BOT_TOKEN = "你的 Bot Token"
TELEGRAM_WEBHOOK_SECRET = "随便一串随机字符"
```

### 第 5 步：部署

```bash
wrangler deploy
```

看到 `Deployed` 和你的 `xxx.workers.dev` 地址就成功了。

### 第 6 步：首次运行

访问下面的地址生成第一期日报（把 `你的令牌` 换成 `CRON_TOKEN` 的值）：

```
https://你的worker地址.workers.dev/cron?token=你的令牌
```

等待约 1 分钟，返回的 JSON 会显示每一步的执行结果。之后系统会**每天自动运行**，无需人工干预。

---

### 可选：配置 Telegram 推送

<details>
<summary>点开查看步骤</summary>

1. 在 Telegram 里找 [@BotFather](https://t.me/BotFather)，发送 `/newbot` 创建机器人，拿到 Token
2. 把 Token 填入 `wrangler.toml` 的 `TELEGRAM_BOT_TOKEN`，重新 `wrangler deploy`
3. 把 Webhook 指向你的 Worker：

```bash
curl -X POST "https://api.telegram.org/bot<你的Token>/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://你的worker地址.workers.dev/telegram/webhook","secret_token":"tg-bind-2026"}'
```

4. **在 Telegram 里给你的机器人发一条消息**（比如 `/start`）——这一步必须手动做，机器人无法主动找你
5. 验证绑定：访问 `https://你的worker地址.workers.dev/telegram/test?token=你的令牌`

</details>

---

## 🔌 API 接口

### 公开接口（无需登录）

| 路径 | 方法 | 说明 |
|---|---|---|
| `/` `/report` | GET | 日报网页，省略 `?date=` 则显示最新一期 |
| `/report?date=2026-10-13` | GET | 查看指定日期 |
| `/rss?days=7` | GET | RSS 订阅源，`days` 控制回看天数 |
| `/getContent?date=2026-10-13` | GET | 返回当日抓取到的原始数据（JSON） |

### 管理接口（需要登录）

| 路径 | 方法 | 说明 |
|---|---|---|
| `/login` | GET/POST | 管理后台登录页 |
| `/getContentHtml` | GET | 日报管理页：勾选条目 → 生成 → 发布 |
| `/writeData` | POST | 手动抓取所有数据源 |
| `/genAIContent` | POST | 根据选中条目生成日报 |
| `/commitToGitHub` | POST | 把日报提交到 GitHub |

### 运维接口（需要 Token）

| 路径 | 说明 |
|---|---|
| `/cron?token=xxx` | 手动跑一次完整流水线 |
| `/telegram/test?token=xxx` | 发一条测试消息 |
| `/telegram/push?date=xxx&token=xxx` | 重推指定日期的日报 |
| `/telegram/bind` | 自动发现并绑定 Telegram chat_id |

---

## 📁 目录结构

```
src/
├── index.js               # Worker 入口，路由分发
├── cron.js                # 定时任务：串起完整流水线
├── chatapi.js             # 大模型调用（含主备自动降级）
├── telegram.js            # Telegram 推送
├── helpers.js             # 日期、转义等工具函数
├── github.js              # GitHub API 封装
├── kv.js                  # KV 读写
├── dataSources/           # 数据源抓取器
│   ├── hackernews.js      #   Hacker News
│   ├── arxiv.js           #   arXiv 论文
│   ├── github-trending.js #   GitHub 项目
│   └── {news,papers,twitter,reddit}.js   # Folo 系（需 Cookie）
└── handlers/              # 各接口的处理逻辑

daily/                     # 每日日报原文（自动提交）
rss/                       # RSS 精简版内容（自动提交）
wrangler.toml              # 部署配置
```

---

## ❓ 常见问题

<details>
<summary><b>为什么日报里只有 GitHub 项目，没有新闻和论文？</b></summary>

说明 Hacker News 和 arXiv 抓取失败了。访问 `/cron?token=xxx` 看返回的 JSON，检查 `writeData` 那一步的 `newsItemCount` 和 `paperItemCount`。

如果都是 0，可能是网络问题或上游 API 变更，查看实时日志：`wrangler tail`。

</details>

<details>
<summary><b>报错 "You exceeded your current quota"</b></summary>

大模型免费额度用完了。Gemini 免费层每天 500 次，北京时间早上 8 点重置。

代码已内置自动降级：Gemini 失败会自动切到 Groq。如果两边都满了，等次日重置即可。

</details>

<details>
<summary><b>日报日期显示 1970-01-01</b></summary>

这是 esbuild 对默认参数 `new Date()` 做 `@__PURE__` 常量折叠导致的（与系统时间无关，也与时区无关）。

本仓库已修复。如果你基于旧版代码部署，更新 `src/helpers.js` 里的 `getISODate` 函数，不要在默认参数里直接写 `new Date()`，改用显式的 `Date.now()`。

</details>

<details>
<summary><b>Telegram 收不到推送</b></summary>

按顺序检查：

1. 有没有**主动给机器人发过消息**？（必须手动做一次）
2. 访问 `/telegram/test?token=xxx` 看返回的 `skipped` 或错误信息
3. 访问 `/telegram/bind` 查看是否检测到会话

</details>

<details>
<summary><b>怎么调整抓取数量？</b></summary>

在 `wrangler.toml` 里调整：

```toml
HN_FILTER_DAYS = "2"      # Hacker News 回看天数
HN_MIN_POINTS = "50"      # 最低热度门槛
ARXIV_MAX_RESULTS = "20"  # arXiv 论文数
GITHUB_AI_TOPICS = "ai,llm,agents,machine-learning,generative-ai"
```

</details>

<details>
<summary><b>怎么换大模型？</b></summary>

改 `wrangler.toml` 里的 `USE_MODEL_PLATFORM`：

- `"GEMINI"` — 用 Gemini 作主模型（`GEMINI_API_KEY`）
- `"OPEN"` — 用 OpenAI 兼容接口作主模型（`OPENAI_API_KEY` + `OPENAI_API_URL`）

只要对方兼容 OpenAI 的 `/v1/chat/completions` 协议（DeepSeek、豆包、OneAPI 中转等都行），改 `OPENAI_API_URL` 和 `DEFAULT_OPEN_MODEL` 即可。

</details>

---

## 🔗 相关仓库

| 仓库 | 作用 |
|---|---|
| [ai-daily-site](https://github.com/siweimidu/ai-daily-site) | 日报展示站（Hugo + Hextra，部署在 GitHub Pages） |
| [justlovemaki/CloudFlare-AI-Insight-Daily](https://github.com/justlovemaki/CloudFlare-AI-Insight-Daily) | 本项目的上游 |

---

## 📄 License

MIT。基于 [justlovemaki/CloudFlare-AI-Insight-Daily](https://github.com/justlovemaki/CloudFlare-AI-Insight-Daily) 修改。
