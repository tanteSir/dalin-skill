# seo-google

> 海外 Google SEO 全链路配置 — 让网站被 Google 收录并获得排名。

适用于面向海外用户（英文市场）的 Next.js (App Router) 项目的 Google SEO 标准化流程。

## 能力一览

| 能力 | 说明 |
|------|------|
| Google Search Console | 资源添加 + sitemap 提交 + 索引请求 |
| Google Analytics 4 | GA4 安装代码模板 |
| 英文 Metadata | 英文 title / description / OG / hreflang 多语言互指 |
| hreflang 标签 | 中英文站双向互指，避免重复内容惩罚 |
| Core Web Vitals | LCP / INP / CLS 优化指引 + Next.js 自动优化 |
| Google Indexing API | 主动推送 URL，200 条/天 |
| JSON-LD 富摘要 | FAQPage / SoftwareApplication / HowTo / BreadcrumbList |
| Sitemap + Robots | 动态生成，只含英文公开页面 |

## 执行清单（7 层）

1. Google Search Console（最先做）
2. Google Analytics（必须）
3. Next.js Metadata（英文）
4. hreflang 标签（关键）
5. Sitemap + Robots
6. JSON-LD 结构化数据
7. Core Web Vitals 优化

## 与 seo-china 的关系

- `seo-china` 负责百度/国内 SEO
- `seo-google` 负责 Google/海外 SEO
- 通用模板（sitemap、robots、JSON-LD）复用 `seo-china/references/`
- 两个站通过 **hreflang 标签**互指，避免重复内容惩罚

## 耗时

约 2h

---

Ideas or suggestions? Let's talk:

WeChat: tanteSir
GitHub Issues: https://github.com/tanteSir/dalin-skills/issues
