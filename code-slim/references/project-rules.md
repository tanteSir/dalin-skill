# 项目级白名单

> 哪些代码看起来像死代码但其实不是，哪些模块/文件不能删。
> 扫描时必须先读此文件，避免误删。

## 通用保护规则

以下类型的代码**禁止删除**，即使 grep 零引用：

1. **框架入口** — `main.py` / `app.py` / `page.tsx` / `layout.tsx` / `__init__.py`
2. **ORM Model** — 所有 SQLAlchemy Model 文件（被 Alembic / metadata 间接引用）
3. **Celery 任务** — `tasks.py` 中的任务函数（被 Celery worker 通过字符串名注册）
4. **配置/常量** — `config.py` / `constants.ts` / `enums.py`（被运行时读取）
5. **Migration 文件** — Alembic / Prisma / Django migration
6. **种子数据** — `seed_*.py` / `fixtures/` 目录
7. **Docker/K8s 配置** — `Dockerfile` / `docker-compose.*` / `nginx/`
8. **Public 静态资源** — `public/` 目录下的文件

## Python 特有保护

| 模式 | 原因 |
|------|------|
| `models/__init__.py` 的 import | SQLAlchemy 需要所有 model 注册到 metadata |
| 被 `@pytest.fixture` 装饰的函数 | pytest 通过装饰器发现 |
| 被 `@app.route` / `@router.get` 等装饰的函数 | 框架通过装饰器注册路由 |
| `if __name__ == "__main__"` 块 | 脚本入口 |
| `conftest.py` 中的 fixture | pytest 自动发现机制 |
| `__all__` 列表中的符号 | 公开 API 声明 |
| 环境变量读取（`os.environ.get`） | 运行时依赖，grep 找不到 |
| 字符串形式的引用（如 `"app.tasks.xxx"`） | Celery task 名称、Dotted paths |

## TypeScript/React 特有保护

| 模式 | 原因 |
|------|------|
| `export default` 的组件 | Next.js 通过文件路径自动路由，无需显式 import |
| `page.tsx` / `layout.tsx` / `loading.tsx` / `error.tsx` | Next.js 约定路由 |
| `middleware.ts` | Next.js 自动加载 |
| CSS Module 文件（`*.module.css`） | 通过 import 间接引用 |
| `public/` 下的静态资源 | 通过 URL 路径引用，grep 找不到 |
| Server Action 函数 | 通过 `"use server"` 标记，被框架发现 |
| API Route (`route.ts`) | Next.js 自动注册 |

## ContentFlow 项目专用保护

以下文件/模块**不能删**，即使看起来没被引用：

| 文件 | 原因 |
|------|------|
| `backend/app/models/__init__.py` | 必须 import 所有 model |
| `backend/app/core/enums.py` | 被多个模块间接引用 |
| `backend/app/core/db_config.py` | 运行时读取 system_config |
| `backend/app/worker/tasks.py` | Celery 通过字符串名发现任务 |
| `backend/scripts/fetch_social_hotspots.py` | 本地 cron 定时执行 |
| `dashboard/src/lib/multipost.ts` | Chrome 扩展通信 |
| `dashboard/messages/zh.json` / `en.json` | i18n 运行时加载 |

## 副作用 import 白名单

以下 import 即使没有在文件中使用也不能删：

```python
# Python — 触发注册/初始化
import matplotlib.pyplot  # 触发后端注册
import structlog  # 触发配置
from app.models import *  # 触发 SQLAlchemy metadata 注册
```

```typescript
// TypeScript — 触发副作用
import './globals.css'  # 样式注入
import '@fontsource/xxx'  # 字体加载
```
