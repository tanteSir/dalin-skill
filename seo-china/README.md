# seo-china

> 国内百度 SEO 全链路配置 — 从零到收录，8 层执行清单。

适用于面向国内用户的 Next.js (App Router) + Nginx 项目的百度 SEO 标准化流程。

## 能力一览

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

## 执行清单（8 层）

1. Next.js Metadata（必须）
2. 页面级 SEO（按需）
3. Sitemap + Robots（必须）
4. JSON-LD 结构化数据（重要）
4.5 FAQ 板块（强烈推荐）
5. 公开内容页（SEO 增长引擎）
6. Nginx 优化（必须）
7. 搜索引擎提交（部署后）
7.5 百度自动推送（持续收录）
8. OG 封面图 + Manifest（锦上添花）

## 参考文档（references/）

| 文件 | 内容 |
|------|------|
| `nextjs-templates.md` | layout / sitemap / robots / manifest / 百度统计代码模板 |
| `nginx-template.md` | 完整 nginx 配置（域名统一 / gzip / 缓存 / 安全头） |
| `jsonld-templates.md` | JSON-LD 模板 + FAQPage 完整模板 |
| `og-image-generation.md` | OG 封面图 3 种生成方式 + prompt 模板 |
| `public-content-page.md` | 公开内容页模板 + 详情页进阶模式 |
| `baidu-auto-push.md` | 百度自动推送 Celery 任务完整代码 |
| `seo-ranking-research.md` | 百度+Google 排名研究数据 |

## 快速开始

1. 将 `seo-china/` 复制到 agent skills 目录
2. 准备前置信息：主域名、百度验证 meta、百度 API token、OG 封面图
3. 按 SKILL.md 的 8 层清单逐步执行

## 耗时

约 2.5h（不含公开内容页约 1.5h）

---

Ideas or suggestions? Let's talk:

WeChat: tanteSir
GitHub Issues: https://github.com/tanteSir/dalin-skills/issues
