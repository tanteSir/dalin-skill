# JSON-LD 结构化数据模板

## 在落地页 page.tsx 中注入

```tsx
export default function LandingPage() {
  const jsonLd = {
    "@context": "https://schema.org",
    "@graph": [
      {
        "@type": "Organization",
        name: "品牌名",
        url: "https://www.example.com",
        description: "一句话描述",
      },
      {
        "@type": "SoftwareApplication",
        name: "产品名",
        url: "https://www.example.com",
        applicationCategory: "Content Creation", // 或其他类别
        operatingSystem: "Web",
        description: "详细描述产品功能",
        offers: [
          { "@type": "Offer", name: "基础版", price: "19", priceCurrency: "CNY", description: "..." },
          { "@type": "Offer", name: "专业版", price: "59", priceCurrency: "CNY", description: "..." },
        ],
      },
      {
        "@type": "WebSite",
        name: "品牌名",
        url: "https://www.example.com",
      },
    ],
  };

  return (
    <main>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* 页面内容 */}
    </main>
  );
}
```

## 常用 schema 类型

| 类型 | 用途 | 适用场景 |
|------|------|----------|
| `Organization` | 公司/组织信息 | 所有企业网站 |
| `SoftwareApplication` | 软件产品信息 | SaaS 产品 |
| `WebSite` | 网站基本信息 | 所有网站 |
| `FAQPage` | FAQ 问答 | 有 FAQ 的页面 |
| `HowTo` | 教程/步骤 | 有教程的页面 |
| `Article` | 文章 | 博客/内容页 |
| `BreadcrumbList` | 面包屑导航 | 有层级结构的页面 |
| `NewsArticle` | 新闻/热点文章 | 内容详情页（热点、资讯等） |

## NewsArticle 结构化数据（内容详情页）

用于热点、资讯、博客等独立内容页。每个详情页是独立的 SEO 入口，帮助搜索引擎展示文章富摘要。

```tsx
// src/app/[content]/[id]/page.tsx
export default async function ContentDetailPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const item = await getItem(id);

  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "NewsArticle",
    headline: item.title,
    description: item.summary || item.title,
    datePublished: item.created_at,
    dateModified: item.created_at,
    author: {
      "@type": "Organization",
      name: "品牌名",
      url: "https://www.example.com",
    },
    publisher: {
      "@type": "Organization",
      name: "品牌名",
      url: "https://www.example.com",
    },
    mainEntityOfPage: {
      "@type": "WebPage",
      "@id": `https://www.example.com/${content}/${id}`,
    },
  };

  return (
    <main>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* 页面内容 */}
    </main>
  );
}
```

### 动态 metadata 配合

```tsx
export async function generateMetadata({ params }: { params: Promise<{ id: string }> }): Promise<Metadata> {
  const { id } = await params;
  const item = await getItem(id);
  const CATEGORY_SUFFIX: Record<string, string> = {
    "科技": "科技行业热点分析",
    "商业": "商业财经选题素材",
    // 按你的分类扩展...
  };
  const suffix = CATEGORY_SUFFIX[item.category || ""] || "内容素材";

  return {
    title: `${item.title} | ${suffix} — 品牌名`,
    description: `${item.summary || item.title}。品牌名精选${item.category || ""}领域内容，提供深度分析。`,
    alternates: { canonical: `https://www.example.com/${content}/${id}` },
    openGraph: {
      title: `${item.title} | ${suffix} — 品牌名`,
      description: `${item.summary || item.title}`,
      url: `https://www.example.com/${content}/${id}`,
    },
  };
}
```

## FAQPage 结构化数据（落地页 FAQ 板块）

落地页加 FAQ 板块可同时提升 SEO（Google 富摘要）和用户信任度。

### 1. 数据定义（module scope，供 JSON-LD 和 JSX 共用）

```tsx
const FAQ_ITEMS = [
  {
    question: "产品是什么？",
    answer: "一句话回答。",
  },
  // 4-8 个 Q&A
];
```

### 2. JSON-LD 注入（加到 `@graph` 数组里）

```tsx
const jsonLd = {
  "@context": "https://schema.org",
  "@graph": [
    // ... Organization, SoftwareApplication, WebSite ...
    {
      "@type": "FAQPage",
      mainEntity: FAQ_ITEMS.map((item) => ({
        "@type": "Question",
        name: item.question,
        acceptedAnswer: {
          "@type": "Answer",
          text: item.answer,
        },
      })),
    },
  ],
};
```

### 3. FAQ 板块 JSX（与现有落地页风格统一）

放在最后一个内容 section 和 Pricing section 之间。

```tsx
<section id="faq" className="border-t border-border bg-[#fbfcfb]">
  <div className="mx-auto flex max-w-[var(--landing-shell-max)] flex-col px-[var(--landing-shell-padding-x)] py-[var(--landing-section-space-y)]">
    <div className="mb-[clamp(2rem,2.8vw,2.75rem)] max-w-4xl">
      <p className="text-[clamp(0.85rem,0.9vw,0.95rem)] uppercase tracking-[0.22em] text-primary">FAQ</p>
      <h2 className="mt-3 text-[length:var(--landing-section-title-size)] font-semibold">常见问题</h2>
    </div>

    <div className="grid gap-[var(--landing-card-gap)] sm:grid-cols-2 lg:grid-cols-3">
      {FAQ_ITEMS.map((item) => (
        <article
          key={item.question}
          className="rounded-[var(--landing-card-radius)] border border-border bg-card p-[var(--landing-card-padding)] shadow-[0_16px_45px_rgba(16,33,30,0.05)]"
        >
          <div className="text-[clamp(1.1rem,1.2vw,1.3rem)] font-semibold leading-[1.45]">{item.question}</div>
          <p className="mt-[clamp(0.6rem,0.8vw,0.75rem)] text-[clamp(0.9rem,0.94vw,0.98rem)] leading-[1.8] text-muted-foreground">{item.answer}</p>
        </article>
      ))}
    </div>
  </div>
</section>
```

### FAQ 内容建议

| Q | 回答方向 |
|---|----------|
| 产品是什么 | 一句话定位 + 核心能力 |
| 谁在背后做 | 创始人/团队信息，增加 E-E-A-T 信号 |
| 和竞品不同 | 差异化卖点，不是聊天框而是工作流 |
| 支持哪些平台 | 具体平台列表或数量 |
| 未来规划 | 路线图简述，展示长期主义 |
| 怎么联系 | 微信/邮箱，降低用户信任门槛 |
