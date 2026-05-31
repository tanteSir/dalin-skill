# 百度 SEO 自动推送

> Celery beat 每日定时推送新增 URL 到百度收录 API。

## 百度配额

| 类型 | 配额 | 说明 |
|------|------|------|
| 普通收录 | 10 条/天 | `http://data.zz.baidu.com/urls` |
| 快速抓取 | 10 条/天 | `http://data.zz.baidu.com/urls?submit=rapid` |

超出配额返回 `{"error": 400, "message": "over quota"}`。

## 1. Settings 新增字段

```python
# app/config.py
class Settings(BaseSettings):
    # ... 已有字段 ...
    
    # --- 百度 SEO ---
    BAIDU_API_TOKEN: str = ""
    BAIDU_SITE: str = "https://www.example.com"
```

`.env` 文件：
```
BAIDU_API_TOKEN=你的百度API_token
BAIDU_SITE=https://www.example.com
```

## 2. 推送脚本

`app/worker/baidu_push.py`：

```python
"""百度 SEO URL 每日推送 — 查询新增内容，推送到百度收录 API。"""

import logging
import httpx
from datetime import datetime, timedelta

from sqlalchemy import select
from app.config import settings
from app.models.hotspot import Hotspot  # 换成你的内容模型

logger = logging.getLogger(__name__)

SITE = settings.BAIDU_SITE
STATIC_URLS = [
    f"{SITE}/",
    f"{SITE}/hotspots",  # 换成你的列表页
]


async def push_urls_to_baidu(urls: list[str]) -> dict:
    if not settings.BAIDU_API_TOKEN:
        logger.warning("BAIDU_API_TOKEN not set, skip push")
        return {"success": 0, "error": "no token"}
    api = f"http://data.zz.baidu.com/urls?site={SITE}&token={settings.BAIDU_API_TOKEN}"
    body = "\n".join(urls)
    async with httpx.AsyncClient(timeout=15) as client:
        resp = await client.post(api, content=body, headers={"Content-Type": "text/plain"})
        result = resp.json()
        logger.info("Baidu push result: %s", result)
        return result


async def get_new_content_urls(hours: int = 24) -> list[str]:
    """查询过去 N 小时新增的内容页 URL。根据你的模型修改。"""
    from app.database import async_session
    cutoff = datetime.utcnow() - timedelta(hours=hours)
    urls = []
    async with async_session() as db:
        rows = await db.execute(
            select(Hotspot.id)
            .where(Hotspot.created_at >= cutoff)
            .order_by(Hotspot.created_at.desc())
            .limit(20)
        )
        for row in rows:
            urls.append(f"{SITE}/hotspots/{row[0]}")
    return urls


async def baidu_daily_push():
    content_urls = await get_new_content_urls()
    # 控制总量不超过配额：静态页 + 内容页 ≤ 10
    max_content = 10 - len(STATIC_URLS)
    all_urls = STATIC_URLS + content_urls[:max_content]
    if not all_urls:
        logger.info("No URLs to push")
        return
    logger.info("Pushing %d URLs to Baidu (static=%d, content=%d)",
                len(all_urls), len(STATIC_URLS), len(content_urls[:max_content]))
    result = await push_urls_to_baidu(all_urls)
    success = result.get("success", 0)
    remain = result.get("remain", 0)
    logger.info("Baidu push done: success=%d, remain=%d", success, remain)
```

## 3. Celery 任务 + Beat 调度

`app/worker/celery_app.py` 新增：

```python
# beat_schedule 新增：
"baidu-daily-push": {
    "task": "app.worker.celery_app.baidu_push_task",
    "schedule": crontab(hour=13, minute=0),  # 每天 13:00
},

# 新增任务：
@celery_app.task(name="app.worker.celery_app.baidu_push_task")
def baidu_push_task():
    from app.worker.baidu_push import baidu_daily_push
    _run_async(baidu_daily_push())
```

## 4. 部署后验证

```bash
# 1. 确认 import 正常
docker compose exec backend python -c "from app.worker.baidu_push import baidu_daily_push; print('OK')"

# 2. 手动触发一次
docker compose exec celery-worker celery -A app.worker.celery_app call app.worker.celery_app.baidu_push_task

# 3. 查看 worker 日志
docker logs celery-worker --tail 10 | grep "Baidu push"

# 4. 或用 curl 直接测试 API
curl -s 'http://data.zz.baidu.com/urls?site=https://你的域名&token=xxx' \
  -H 'Content-Type:text/plain' -d 'https://你的域名/'
# 期望: {"remain":9,"success":1}
```

## URL 优先级策略

配额有限（10 条/天），按优先级推送：

1. **静态页面**（首页、列表页）— 每次都推，确保收录稳定
2. **最新内容页** — 按时间倒序取最新
3. **不要重复推相同 URL** — 百度会去重但不扣配额，但也没额外效果

## 坑点

| 问题 | 原因 | 解法 |
|------|------|------|
| `over quota` | 当天配额用完 | 控制 URL 数量 ≤ 10；或改用快速抓取接口 |
| beat 修改 schedule 不生效 | beat 容器用的旧代码 | 重启 beat 容器：`docker compose restart celery-beat` |
| `asyncio.run()` 测试报错 | event loop 冲突 | 用 `celery call` 或手动 curl 测试，不要用 `asyncio.run()` |
| 推送成功但不收录 | 百度收录需要时间 | 正常，通常 1-7 天；确保页面有真实内容 |
