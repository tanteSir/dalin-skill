# Next.js Metadata 配置模板

## layout.tsx 根 metadata

```typescript
import type { Metadata } from "next";

const SITE_URL = "https://www.example.com"; // 用户确认的主域名

export const metadata: Metadata = {
  title: "产品名 — 一句话定位（含核心关键词）",
  description: "150字以内中文描述，重复核心关键词2-3次，说明产品价值",
  keywords: ["关键词1", "关键词2", ...], // 10-15个，包含行业词+产品词
  authors: [{ name: "品牌名" }],
  metadataBase: new URL(SITE_URL),
  alternates: { canonical: SITE_URL },
  openGraph: {
    type: "website",
    locale: "zh_CN",
    url: SITE_URL,
    siteName: "品牌名",
    title: "品牌名 — 一句话定位",
    description: "分享到社交媒体时显示的描述",
    images: [{ url: "/og-image.jpg", width: 1200, height: 630, alt: "品牌名" }],
  },
  twitter: {
    card: "summary_large_image",
    title: "...",
    description: "...",
    images: ["/og-image.jpg"],
  },
  verification: {
    other: { "baidu-site-verification": "用户提供的验证码" },
  },
  robots: { index: true, follow: true },
};
```

## 百度统计安装

在 `layout.tsx` 的 `<body>` 内添加百度统计脚本（增加百度信任信号）：

```typescript
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body>
        {/* 百度统计 — 用你的 key 替换 */}
        <script dangerouslySetInnerHTML={{
          __html: `
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?你的百度统计key";
  var s = document.getElementsByTagName("script")[0];
  s.parentNode.insertBefore(hm, s);
})();
          `,
        }} />
        {children}
      </body>
    </html>
  );
}
```

百度统计 key 获取：[百度统计](https://tongji.baidu.com) → 管理 → 获取代码 → 提取 `hm.js?` 后面的字符串。

## 页面级 metadata

```typescript
// src/app/page.tsx（落地页）
export const metadata: Metadata = {
  title: "更具体的页面标题 | 品牌名",
  description: "这个页面特有的描述",
  alternates: { canonical: "https://www.example.com/" },
  openGraph: {
    title: "...",
    description: "...",
    url: "https://www.example.com/",
  },
};
```

## sitemap.ts

```typescript
import type { MetadataRoute } from "next";

const SITE_URL = "https://www.example.com";

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: SITE_URL, lastModified: new Date(), changeFrequency: "weekly", priority: 1.0 },
    // 其他公开页面...
    // 注意：不要加入需要登录的页面
  ];
}
```

## robots.ts

```typescript
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [{
      userAgent: "*",
      allow: "/",
      disallow: ["/api/", /* 其他需要登录的路径 */],
    }],
    sitemap: "https://www.example.com/sitemap.xml",
  };
}
```

## manifest.ts

```typescript
import type { MetadataRoute } from "next";

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "产品名 — 一句话定位",
    short_name: "产品名",
    description: "...",
    start_url: "/workspace",
    display: "standalone",
    background_color: "#背景色",
    theme_color: "#主色",
    icons: [{ src: "/favicon.ico", sizes: "any", type: "image/x-icon" }],
  };
}
```
