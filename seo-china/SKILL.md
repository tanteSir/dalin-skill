---
name: seo-china
description: 国内百度SEO全链路配置。包括百度站长验证、API推送、百度统计、中文metadata、sitemap/robots、JSON-LD结构化数据、FAQ板块、公开内容页SSR、百度自动推送、Nginx优化。关键词：国内SEO、百度SEO、百度收录、百度站长、百度统计、SEO优化、sitemap、robots、结构化数据、ICP备案
---

# 国内 SEO 全链路配置 — 让网站被百度收录并获得排名

> Next.js (App Router) + Nginx 项目的百度 SEO 标准化流程。
> 适用于面向国内用户的 SaaS / 内容站点。
> 海外版 SEO 见 `seo-google` skill。

## 触发词

国内SEO、百度SEO、百度收录、百度站长、百度统计、SEO优化、sitemap、robots、结构化数据、OG 封面图、排名优化、薄内容、外链、ICP备案

## 前置确认（执行前必须问用户）

| 确认项 | 说明 | 来源 |
|--------|------|------|
| 主域名 | www 还是非 www？统一一个，另一个 301 | 用户决定 |
| 百度验证 meta | `<meta name="baidu-site-verification" content="xxx" />` | [百度搜索资源平台](https://ziyuan.baidu.com) → 添加站点 → HTML 标签验证 |
| 百度 API token | URL 里 `token=xxx` 那串 | 百度站长 → 普通收录 → API 提交 |
| OG 封面图 | 有现成的提供路径，没有就生成 | — |

## 执行清单（8 层，按顺序）

### 1. Next.js Metadata（必须）

修改 `src/app/layout.tsx`，设置中文 title/description/keywords/Open Graph/Twitter Card/百度验证/canonical。

→ 代码模板见 [references/nextjs-templates.md](references/nextjs-templates.md)

### 2. 页面级 SEO（按需）

每个**公开页面**导出独立 `metadata`，覆盖 layout 的通用描述。

→ 代码模板见 [references/nextjs-templates.md](references/nextjs-templates.md)

### 3. Sitemap + Robots（必须）

- `src/app/sitemap.ts` — 动态生成，只放公开页面（不放登录后页面）
- `src/app/robots.ts` — 允许所有爬虫，disallow `/api/` 和登录后路径，指向 sitemap

→ 代码模板见 [references/nextjs-templates.md](references/nextjs-templates.md)

### 4. 结构化数据 JSON-LD（重要）

在落地页注入 `Organization` + `SoftwareApplication` + `WebSite`，帮助搜索引擎展示富摘要。

如果有 FAQ 板块，额外加 `FAQPage` schema — Google 会在搜索结果中展示 FAQ 富摘要（可展开的问答）。

→ 代码模板见 [references/jsonld-templates.md](references/jsonld-templates.md)

### 4.5 FAQ 板块（强烈推荐）

落地页加 FAQ 板块，一石三鸟：
1. **SEO** — `FAQPage` JSON-LD → Google 富摘要（搜索结果中展开问答）
2. **E-E-A-T** — 创始人信息、联系方式 → 搜索引擎信任信号
3. **转化** — 解答常见疑虑，降低注册门槛

实现要点：
- 数据定义为 module scope 常量（`FAQ_ITEMS`），供 JSON-LD 和 JSX 共用
- 布局用 card grid（与现有落地页风格统一），不用 accordion
- 放在内容板块和 Pricing 之间
- 6 个 Q&A 为佳（覆盖：产品是什么 / 谁在做 / 差异化 / 平台支持 / 未来规划 / 联系方式）

→ 完整模板见 [references/jsonld-templates.md](references/jsonld-templates.md) 的「FAQPage 结构化数据」章节

### 5. 公开内容页（SEO 增长引擎）

如果有持续更新的内容（热点/博客/案例），开设不需要登录的公开页面是 SEO 最有效的手段。

关键决策：
- 页面放在 `src/app/` 下（不在 `src/app/workspace/` 下），绕过 auth layout
- `export const dynamic = "force-dynamic"` — 避免构建时预渲染空数据
- 后端新增公开端点（不依赖 `get_current_user`）
- SSR fetch 用 Docker 内部网络地址（`http://backend:8000`），不是 `localhost`

→ 完整模板见 [references/public-content-page.md](references/public-content-page.md)

### 6. Nginx 优化（必须）

三个改动：
1. **域名统一**：非主域名 301 → 主域名（避免重复内容惩罚）
2. **Gzip 压缩**：减少传输体积，提升 Core Web Vitals
3. **安全头 + 静态资源 30 天缓存**

→ 配置模板见 [references/nginx-template.md](references/nginx-template.md)

### 7. 搜索引擎提交（部署后）

**百度**（API 推送）：
```bash
curl 'http://data.zz.baidu.com/urls?site=https://域名&token=xxx' \
  -H 'Content-Type:text/plain' -d 'https://域名/每个公开URL'
```

**Google**：手动去 [Search Console](https://search.google.com/search-console) 添加资源 → 提交 sitemap URL。

### 7.5 百度自动推送（持续收录，强烈推荐）

手动推送只能用一次。要让新增内容页持续被收录，需要 **Celery beat 每日定时推送**。

核心逻辑：
1. 查询过去 24h 新增的公开内容页（热点/博客等）
2. 加上静态页面（首页、列表页）
3. 调百度 API 批量推送
4. 百度配额：**10 条/天**（普通收录），超出返回 400 `over quota`

需要环境变量：
- `BAIDU_API_TOKEN` — 百度站长 API token
- `BAIDU_SITE` — 主域名（如 `https://www.ipworkhub.cn`）

触发时间建议 13:00（工作时间，便于排查问题）。

→ 完整代码模板见 [references/baidu-auto-push.md](references/baidu-auto-push.md)

### 8. OG 封面图 + Manifest（锦上添花）

- OG 封面图：1200×630 JPG（<100KB），用 GPT Image 2 / Playwright / SVG 三种方式生成
- Manifest：`src/app/manifest.ts`，PWA 配置

→ 生成指南见 [references/og-image-generation.md](references/og-image-generation.md)
→ Manifest 模板见 [references/nextjs-templates.md](references/nextjs-templates.md)

## 检查清单（完成后逐项验证）

- [ ] `curl https://域名/` HTML 包含中文 title、description、og:title、baidu-site-verification
- [ ] `curl https://域名/robots.txt` 正确的 Allow/Disallow + Sitemap 行
- [ ] `curl https://域名/sitemap.xml` 包含所有公开页面 URL
- [ ] 百度站长验证通过
- [ ] 百度 API 推送返回 `{"success": N}`
- [ ] Google Search Console 已添加资源
- [ ] `curl https://非主域名/` 返回 301 到主域名
- [ ] 公开内容页 SSR 渲染真实数据（curl 能看到内容文字，不是空白）
- [ ] OG 封面图可访问（`curl https://域名/og-image.jpg` 返回 200，<100KB）
- [ ] 落地页 HTML 包含 FAQPage JSON-LD（`curl | grep FAQPage`）
- [ ] FAQ 板块可见且风格与其他 section 统一
- [ ] `docker logs celery-worker | grep "Baidu push"` 确认每日推送执行（如有自动推送）

## 坑点速查

| 坑 | 解法 |
|----|------|
| Next.js standalone 构建时无后端，SSR fetch 失败 | `export const dynamic = "force-dynamic"` + Docker 内部网络地址 |
| Docker 容器内 SSR fetch localhost 不通 | 用 `http://backend:8000`（compose service name） |
| 百度验证"无法连接" | 先部署代码再点验证；确保 nginx 不 502 |
| 重复提交相同 URL | 百度会去重，不扣配额；但也没额外效果 |
| `NEXT_PUBLIC_` 变量构建时内联 | 非 public 变量 standalone 运行时读不到，硬编码内部地址 |
| 搜索引擎抓到空页面 | 确认 curl 能看到真实文字（不是 JS 渲染） |
| OG 封面图太大（>200KB） | JPEG quality=85 压缩，目标 <100KB；避免 PNG |
| OG 图在微信不显示 | 检查图片 URL 是否公网可访问；`og:image` 必须是完整 HTTPS URL |
| 百度推送 `over quota` | 普通收录配额 10 条/天，超出返回 400；控制推送 URL 数量 |
| Celery beat 修改 schedule 不生效 | 需重启 beat 容器（`docker compose restart celery-beat`），旧容器用的是旧代码 |
| `asyncio.run()` 测试报 event loop 警告 | 不影响 Celery `_run_async`；如需测试用手动 curl 或 `celery call` |

## 耗时参考

| 步骤 | 耗时 |
|------|------|
| Next.js metadata + sitemap + robots | 30min |
| Nginx 优化 | 15min |
| JSON-LD 结构化数据 | 15min |
| FAQ 板块（含 FAQPage schema） | 20min |
| 公开内容页（如果有） | 45min |
| OG 封面图生成 | 15min |
| 百度站长验证 + 提交 | 15min |
| 百度自动推送 | 15min |
| Google Search Console | 15min |
| **总计** | **~2.5h** |

## 排名优化（收录后的进阶工作）

基础 SEO 完成后，排名优化是持续工作。关键杠杆（按权重排序）：

1. **外链** — 6 个高质量外链 > 100 个低质量。知乎/V2EX/少数派发帖带链接。不要买外链。
2. **内容质量** — 每个独立 URL 至少 300-500 字原创内容。薄内容页会被百度细雨算法降权。
3. **程序化页面** — 热点独立页必须有独特价值（不能只是标题+摘要），每页加"AI 分析"或"相关内容"。
4. **E-E-A-T 信号** — 作者署名 + FAQ 板块（含创始人信息+联系方式） + 关于页面。
5. **百度统计** — 安装代码增加百度信任信号。
6. **内链** — Hub-Spoke 模型，相关内容互相链接。
7. **内容新鲜度** — 核心页面每 3-6 个月更新；百度对时效性权重 > Google。

→ 完整研究数据见 [references/seo-ranking-research.md](references/seo-ranking-research.md)
