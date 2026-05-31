# dalin-skills

> A collection of AI agent skills for OpenCode / Claude Code / Codex, built by [dalin](https://github.com/tanteSir).

AI Agent 技能集合，为 OpenCode / Claude Code / Codex 等 AI 编程助手提供专业化能力。

---

## Skills

### [code-slim](./code-slim/)

通用代码瘦身引擎。语言无关，扫描 → 诊断 → 执行 → 验证。

| 能力 | 说明 |
|------|------|
| 死代码检测 | S0 零引用自动删除（文件/函数/变量/常量/未使用 import） |
| 重复代码提取 | S1 3+ 次重复模式提取为共享函数 |
| God 文件拆分 | S2 >500 行多职责文件拆分建议 |
| 安全分级 | S0~S3 四级，逐条执行 + 每条验证 |
| 分支隔离 | 所有改动在 `*-slim` 分支，不污染主线 |
| 全语言支持 | Python / TypeScript / Go / Java / Rust 均适用 |

**触发词**: 代码瘦身、瘦身扫描、slim、精简代码、code slim、trim code

---

### [seo-china](./seo-china/)

国内百度 SEO 全链路配置。从零到收录，8 层执行清单。

| 能力 | 说明 |
|------|------|
| 百度站长验证 | HTML meta 验证 + API 推送 |
| 中文 Metadata | title / description / keywords / OG 全套 / 百度验证 |
| Sitemap + Robots | 动态生成，只含公开页面 |
| JSON-LD 结构化数据 | Organization / SoftwareApplication / WebSite / FAQPage |
| FAQ 板块 | FAQPage schema → Google 富摘要 + E-E-A-T 信号 + 转化提升 |
| 公开内容页 | SSR + Docker 内部网络 + LLM 懒生成 AI 分析 |
| 百度自动推送 | Celery beat 每日推送新增 URL，持续收录 |
| 百度统计 | 安装代码增加百度信任信号 |
| Nginx 优化 | 域名统一 301 / gzip / 安全头 / 静态资源缓存 |

**触发词**: 国内SEO、百度SEO、百度收录、百度站长、百度统计、sitemap、结构化数据

**References**: `nextjs-templates.md` / `nginx-template.md` / `jsonld-templates.md` / `og-image-generation.md` / `public-content-page.md` / `baidu-auto-push.md` / `seo-ranking-research.md`

---

### [seo-google](./seo-google/)

海外 Google SEO 全链路配置。面向英文市场的搜索引擎优化。

| 能力 | 说明 |
|------|------|
| Google Search Console | 资源添加 + sitemap 提交 + 索引请求 |
| Google Analytics 4 | GA4 安装代码模板 |
| 英文 Metadata | 英文 title / description / OG / hreflang 多语言互指 |
| hreflang 标签 | 中英文站双向互指，避免重复内容惩罚 |
| Core Web Vitals | LCP / INP / CLS 优化指引 + Next.js 自动优化 |
| Google Indexing API | 主动推送 URL，200 条/天（百度仅 10 条） |
| JSON-LD 富摘要 | FAQPage / SoftwareApplication / HowTo / BreadcrumbList |

**触发词**: 海外SEO、Google SEO、Google收录、Search Console、Google Analytics、hreflang、Core Web Vitals

---

## 安装使用

将需要的 skill 目录复制到你的 agent skills 目录：

```bash
# OpenCode
cp -r seo-china/ ~/.config/opencode/skills/

# Claude Code
cp -r seo-china/ ~/.claude/skills/

# Codex
cp -r seo-china/ ~/.codex/skills/
```

每个 skill 的 `SKILL.md` 包含完整的执行流程、代码模板、坑点速查和检查清单。

## License

MIT

---

Ideas or suggestions? Let's talk:

WeChat: tanteSir
GitHub Issues: https://github.com/tanteSir/dalin-skills/issues
