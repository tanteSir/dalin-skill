# 公开内容页模板（SSR + Docker 内部网络）

## 前端：Server Component + force-dynamic

```typescript
// src/app/[content-page]/page.tsx
import type { Metadata } from "next";

export const dynamic = "force-dynamic"; // 关键！避免构建时预渲染空数据
export const revalidate = 300; // 5分钟 ISR（可选）

export const metadata: Metadata = {
  title: "页面标题 — 品牌名 | SEO 关键词",
  description: "页面描述",
  alternates: { canonical: "https://www.example.com/page" },
};

async function getData() {
  // Docker 内部网络地址，不是 localhost
  const res = await fetch("http://backend:8000/api/xxx/public", {
    next: { revalidate: 300 },
  });
  if (!res.ok) return [];
  const data = await res.json();
  return data.items || [];
}

export default async function PublicPage() {
  const items = await getData();
  return (
    <main>
      {items.map(item => <article key={item.id}>...</article>)}
    </main>
  );
}
```

## 后端：公开端点（不依赖 auth）

```python
@router.get("/public")  # 不加 user: AuthUser = Depends(get_current_user)
async def content_public(db: AsyncSession = Depends(get_db)):
    cutoff = datetime.now() - timedelta(days=3)
    rows = (await db.execute(
        select(Model)
        .where(Model.created_at >= cutoff, Model.score >= 80)
        .order_by(Model.score.desc())
        .limit(20)
    )).scalars().all()
    return {"items": [item_to_dict(h) for h in rows], "count": len(rows)}
```

## 路由位置

公开页面放在 `src/app/` 下（不在 `src/app/workspace/` 下），避免被 workspace layout 的 auth 拦截。

```
src/app/
├── page.tsx              # 落地页（公开）
├── hotspots/page.tsx     # 公开内容列表页
├── hotspots/[id]/page.tsx # 内容详情页（SEO 增长引擎）
├── login/page.tsx        # 登录页（公开）
└── workspace/            # 受 auth 保护
    ├── layout.tsx        # auth 拦截在这里
    └── ...
```

## 详情页进阶模式（SEO 增长引擎）

每个内容详情页都是独立的 SEO 入口，需要有**独特价值**（不能只是标题+摘要），否则百度细雨算法会降权。

### 详情页 SEO 必备要素

1. **独立 metadata** — 每页有自己的 title（内容标题 — 品牌名）和 description
2. **AI 分析/深度内容** — `writing_analysis` 字段：LLM 懒生成 + DB 缓存
   - 首次访问触发 LLM 生成（~2-3s），后续从 DB 读取
   - 每篇 300-400 字，提供独特价值
3. **相关内容推荐** — 增加内链（Hub-Spoke 模型）
   - 同分类优先（≥3 条），不够则 fallback 到全局热门
4. **来源声明** — "本页内容由AI从公开信息中精选...内容版权归原作者所有"
5. **CTA 引导** — "立即注册，用AI分析更多热点" → 引导注册

### AI 分析懒生成模式（后端）

```python
@router.get("/public/{item_id}")
async def public_detail(item_id: str, db: AsyncSession = Depends(get_db)):
    item = (await db.execute(select(Model).where(Model.id == item_id))).scalar_one_or_none()
    if not item:
        raise HTTPException(404)
    
    # 懒生成 AI 分析（首次访问时生成，后续从 DB 缓存）
    if not item.writing_analysis:
        item.writing_analysis = await _generate_analysis(item.title, item.summary)
        db.add(item)
        await db.commit()
        await db.refresh(item)
    
    return {"item": item_to_dict(item)}

async def _generate_analysis(title: str, summary: str) -> str:
    """用 LLM 生成 300-400 字的写作角度分析，缓存到 DB。"""
    prompt = f"针对以下热点，从内容创作者角度分析写作角度和价值（300-400字）：\n标题：{title}\n摘要：{summary}"
    resp = await ai_client.chat.completions.create(
        model="deepseek-v4-flash",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=600,
    )
    return resp.choices[0].message.content
```

### 相关内容推荐（后端）

```python
@router.get("/public/{item_id}/related")
async def related_items(item_id: str, db: AsyncSession = Depends(get_db)):
    item = (await db.execute(select(Model).where(Model.id == item_id))).scalar_one_or_none()
    if not item:
        return []
    
    # 同分类优先
    same_cat = (await db.execute(
        select(Model).where(Model.category == item.category, Model.id != item_id)
        .order_by(Model.final_score.desc()).limit(3)
    )).scalars().all()
    
    if len(same_cat) >= 3:
        return same_cat
    
    # 不够则补全局热门
    fallback = (await db.execute(
        select(Model).where(Model.id != item_id)
        .order_by(Model.final_score.desc()).limit(5)
    )).scalars().all()
    
    return list(same_cat) + [f for f in fallback if f not in same_cat][:5]
```

### Sitemap 集成

详情页 URL 需要加入 sitemap，后端提供公开 ID 列表：

```python
@router.get("/public-ids")
async def public_ids(db: AsyncSession = Depends(get_db)):
    rows = (await db.execute(
        select(Model.id).where(Model.final_score >= 80)
    )).scalars().all()
    return [str(r) for r in rows]
```

```typescript
// src/app/sitemap.ts
async function getContentIds() {
  const res = await fetch("http://backend:8000/api/xxx/public-ids", {
    next: { revalidate: 3600 },
  });
  return res.ok ? await res.json() : [];
}

export default async function sitemap() {
  const ids = await getContentIds();
  return [
    { url: SITE_URL, lastModified: new Date(), priority: 1.0 },
    { url: `${SITE_URL}/hotspots`, lastModified: new Date(), priority: 0.9 },
    ...ids.map((id: string) => ({
      url: `${SITE_URL}/hotspots/${id}`,
      lastModified: new Date(),
      priority: 0.8,
    })),
  ];
}
```
