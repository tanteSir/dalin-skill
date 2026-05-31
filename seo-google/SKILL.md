---
name: seo-google
description: 海外Google SEO全链路配置。包括Google Search Console、Google Analytics 4、英文metadata、hreflang多语言标签、sitemap/robots、JSON-LD结构化数据、Core Web Vitals优化、Google Indexing API。关键词：海外SEO、Google SEO、Google收录、Search Console、Google Analytics、英文SEO、hreflang、Core Web Vitals、Google排名
---

# 海外 SEO 全链路配置 — 让网站被 Google 收录并获得排名

> Next.js (App Router) 项目的 Google SEO 标准化流程。
> 适用于面向海外用户（英文市场）的 SaaS / 内容站点。
> 国内版 SEO 见 `seo-china` skill。

## 触发词

海外SEO、Google SEO、Google收录、Search Console、Google Analytics、英文SEO、sitemap、robots、结构化数据、Core Web Vitals、hreflang、外链建设、Google排名

## 前置确认（执行前必须问用户）

| 确认项 | 说明 | 来源 |
|--------|------|------|
| 域名 | 海外版域名（如 `.com`） | 用户决定 |
| Google 账号 | 用于 Search Console + Analytics | 用户注册 |
| 部署平台 | Vercel / AWS / Cloudflare Pages | 用户决定 |
| OG 封面图 | 有现成的提供路径，没有就生成 | — |

## 执行清单（7 层，按顺序）

### 1. Google Search Console（最先做）

1. 去 [Search Console](https://search.google.com/search-console) 添加资源（域名验证推荐 DNS 验证）
2. 提交 sitemap URL（如 `https://example.com/sitemap.xml`）
3. 等待 Google 抓取（通常 1-3 天开始收录）

### 2. Google Analytics（必须）

安装 Google Analytics 4（GA4）跟踪代码：

```typescript
// layout.tsx 的 <body> 内
<script
  src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"
  async
/>
<script dangerouslySetInnerHTML={{
  __html: `
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
gtag('js', new Date());
gtag('config', 'G-XXXXXXXXXX');
  `,
}} />
```

GA4 Measurement ID 获取：Google Analytics → 管理 → 数据流 → 复制 Measurement ID。

### 3. Next.js Metadata（英文）

```typescript
// layout.tsx
export const metadata: Metadata = {
  title: "Product Name — One-line positioning with core keywords",
  description: "150 chars English description. Repeat core keywords 2-3 times. Explain product value.",
  keywords: ["keyword1", "keyword2", ...], // 10-15 industry + product keywords
  metadataBase: new URL("https://example.com"),
  alternates: {
    canonical: "https://example.com",
    languages: {
      "en": "https://example.com",
      "zh": "https://www.ipworkhub.cn",  // hreflang 指向中文版
    },
  },
  openGraph: {
    type: "website",
    locale: "en_US",
    url: "https://example.com",
    siteName: "Product Name",
    title: "Product Name — One-line positioning",
    description: "Description shown when shared on social media",
    images: [{ url: "/og-image.jpg", width: 1200, height: 630, alt: "Product Name" }],
  },
  twitter: {
    card: "summary_large_image",
    title: "...",
    description: "...",
    images: ["/og-image.jpg"],
  },
  robots: { index: true, follow: true },
};
```

### 4. hreflang 标签（关键）

**中文站和英文站必须互相指向**，否则 Google 会视为重复内容：

```typescript
// 英文站 layout.tsx
alternates: {
  canonical: "https://example.com",
  languages: {
    "en": "https://example.com",
    "zh": "https://www.ipworkhub.cn",
  },
}

// 中文站 layout.tsx（已有的 seo-china 配置里加）
alternates: {
  canonical: "https://www.ipworkhub.cn",
  languages: {
    "zh": "https://www.ipworkhub.cn",
    "en": "https://example.com",
  },
}
```

### 5. Sitemap + Robots

与国内版相同，参考 `seo-china` skill。区别：
- 英文站的 sitemap 只包含英文页面 URL
- robots.txt 的 Sitemap 指向英文站域名

→ 代码模板见 `seo-china/references/nextjs-templates.md`（通用）

### 6. JSON-LD 结构化数据

Google 对富摘要支持比百度好很多，重点做：

| Schema 类型 | Google 富摘要效果 |
|-------------|------------------|
| `FAQPage` | 搜索结果中展开 Q&A |
| `SoftwareApplication` | 显示评分、价格 |
| `Organization` | 知识面板 |
| `HowTo` | 步骤列表 |
| `BreadcrumbList` | 面包屑导航 |

→ 代码模板见 `seo-china/references/jsonld-templates.md`（通用）

### 7. Core Web Vitals 优化

Google 排名的重要因素，用 [PageSpeed Insights](https://pagespeed.web.dev/) 测试。

| 指标 | 目标 | 优化手段 |
|------|------|---------|
| LCP（最大内容绘制） | <2.5s | 图片 WebP/AVIF、CDN、预加载关键资源 |
| INP（交互延迟） | <200ms | 减少主线程 JS、代码分割 |
| CLS（布局偏移） | <0.1 | 图片设 width/height、字体 `font-display: swap` |

Next.js 自动优化：
- `next/image` — 自动 WebP、懒加载、响应式
- `next/font` — 字体优化，零 CLS
- 代码分割 — 自动按路由拆分

## Google Indexing API（可选，主动推送）

如果需要加速收录（类似百度的 API 推送）：

1. 去 [Google API Console](https://console.cloud.google/) 启用 Indexing API
2. 创建服务账号，获取 JSON 密钥
3. 将 Search Console 所有者权限授予服务账号邮箱

```bash
# 推送单个 URL
curl -X POST "https://indexing.googleapis.com/v3/urlNotifications:publish" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
  -d '{"url": "https://example.com/page", "type": "URL_UPDATED"}'
```

配额：**200 条/天**，远大于百度的 10 条/天。

## 检查清单

- [ ] Google Search Console 已添加资源并验证
- [ ] Sitemap 已提交到 Search Console
- [ ] Google Analytics 代码已安装（`curl | grep googletagmanager`）
- [ ] `curl https://域名/` HTML 包含英文 title、description、og:title
- [ ] hreflang 标签正确指向中文站（`curl | grep hreflang`）
- [ ] PageSpeed Insights 三个核心指标全部绿色
- [ ] 公开内容页 SSR 渲染真实数据（curl 能看到内容文字）
- [ ] OG 封面图可访问（`curl https://域名/og-image.jpg` 返回 200，<100KB）
- [ ] 落地页 HTML 包含 JSON-LD（`curl | grep schema.org`）

## 坑点速查

| 坑 | 解法 |
|----|------|
| Google 收录慢（1-2 周） | 正常；用 Search Console 的"请求索引"加速单个页面 |
| hreflang 不生效 | 确保双向互指（中文站→英文站 + 英文站→中文站） |
| 两个域名内容相同被判定重复 | hreflang + 独立英文内容（不要机翻，Google 能识别） |
| Vercel 部署 SSR 抓取超时 | Vercel Serverless Function 限制 10s；复杂 SSR 用 ISR 或 SSG |
| GA4 数据延迟 24-48h | 正常；实时报告在"报告 → 实时"中查看 |
| Core Web Vitals LCP 差 | 图片用 WebP、上 CDN、`next/image` 自动优化 |

## 排名优化（收录后的进阶工作）

1. **外链** — 英文市场高质量外链渠道：Product Hunt 发布、HackerNews 投稿、Reddit 相关 subreddit、Medium 博客、GitHub 开源项目
2. **内容质量** — 每个独立 URL 至少 500 字原创英文内容
3. **E-E-A-T** — About 页面 + 团队介绍 + 作者署名
4. **内链** — Hub-Spoke 模型，相关内容互相链接
5. **更新频率** — Google 偏好持续更新的站点
6. **页面体验** — Core Web Vitals 全绿 + HTTPS + 无侵入式广告

## 耗时参考

| 步骤 | 耗时 |
|------|------|
| Google Search Console 设置 | 15min |
| Google Analytics 安装 | 15min |
| 英文 Metadata + hreflang | 20min |
| Sitemap + Robots | 15min |
| JSON-LD 结构化数据 | 15min |
| Core Web Vitals 优化 | 30min |
| **总计** | **~2h** |
