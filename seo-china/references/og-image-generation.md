# OG 封面图生成指南

## 方式一：GPT Image 2（推荐）

### Prompt 模板

```
Create a professional Open Graph cover image (1200x630 aspect ratio) for a SaaS product.

Background: [浅色/深色] with subtle gradient accents in [品牌主色 HEX].

Left side:
- Large bold product name "[品牌名]" in [文字色]
- Subtitle "[一句话定位]" in [副色]
- Row of capsule/pill tags: [平台1] [平台2] [平台3]
- Domain URL at bottom

Right side:
- [描述右侧视觉：产品截图抽象化/工作流图标/抽象图形]

Style: Minimal, professional. Clean vector-style. Premium SaaS product feeling.
All text must be in [中文/English].
```

### 调用代码（复用项目 ImageAPI）

```python
"""生成 OG 封面图，保存到 public/og-image.jpg。

用法: cd backend && source .venv/bin/activate && python scripts/generate_og_image.py
"""
from __future__ import annotations
import asyncio, sys, httpx
from pathlib import Path
sys.path.insert(0, str(Path(__file__).resolve().parent.parent))
from app.modules.illustrator.image_api import ImageAPI
from PIL import Image

OG_PROMPT = """..."""  # 填入上面的 prompt
OUTPUT = Path("../dashboard/public/og-image.jpg")

async def main():
    api = ImageAPI()
    result = await api.generate(prompt=OG_PROMPT, size="1792x1024", model="gpt-image-2", max_retries=2)
    url = result.get("url", "")
    if not url:
        print(f"失败: {result.get('error')}"); sys.exit(1)
    async with httpx.AsyncClient(timeout=60) as client:
        img_bytes = (await client.get(url)).content
    img = Image.open(BytesIO(img_bytes)).resize((1200, 630), Image.LANCZOS)
    img.convert("RGB").save(str(OUTPUT), "JPEG", quality=85, optimize=True)
    print(f"已保存: {OUTPUT} ({OUTPUT.stat().st_size / 1024:.0f}KB)")

if __name__ == "__main__":
    asyncio.run(main())
```

## 方式二：HTML + Playwright 截图

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1200, "height": 630})
    page.set_content("""<!DOCTYPE html><html>
      <body style="margin:0;width:1200px;height:630px;background:#F5F8F4;display:flex;...">
        <h1>品牌名</h1>
        <p>一句话定位</p>
      </body></html>""")
    page.screenshot(path="public/og-image.jpg", type="jpeg", quality=85)
    browser.close()
```

## 方式三：SVG 转 PNG

```bash
rsvg-convert -w 1200 -h 630 og-image.svg -o public/og-image.png
# 再转 JPG 减小体积
python3 -c "from PIL import Image; Image.open('public/og-image.png').convert('RGB').save('public/og-image.jpg', 'JPEG', quality=85)"
```

## 优化必做

```python
from PIL import Image
img = Image.open("og-image.png").resize((1200, 630), Image.LANCZOS)
img.convert("RGB").save("public/og-image.jpg", "JPEG", quality=85, optimize=True)
# 目标: <100KB
```
