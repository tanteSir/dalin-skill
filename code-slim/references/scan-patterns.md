# 扫描模式清单

> 静态分析可识别的代码问题模式。扫描时逐项检查，避免遗漏。

## P1: 死代码（零引用，S0）

### 文件级
- 整个文件没有被任何地方 import / require
- 文件存在但 `__init__.py` / `index.ts` 没有 re-export，也没有直接 import

### 函数/方法级
- `def` / `async def` / `function` 定义了但全项目零调用
- Python: 注意 `@pytest.fixture` / `@app.route` 等装饰器注册的函数不算死代码
- TypeScript: 注意 `export` 但从未 import 的函数

### 类级
- `class` 定义了但从未实例化或继承
- Python: 注意 SQLAlchemy Model（通过 `__init__.py` 导入给 Alembic 用）不算死代码
- 注意 dataclass / Pydantic model 可能被类型注解引用

### 常量/变量级
- 模块级常量（全大写 / `const`）定义了但未使用
- TypeScript: `import { X }` 但 X 未在文件中使用

### Import 级
- Python: `import xxx` 但 xxx 未在文件中使用
- TypeScript: `import { X } from 'y'` 但 X 未使用
- 注意：副作用 import（如 `import matplotlib` 触发注册）不算死代码

## P2: 重复代码（≥3 次相似，≥5 行，S1）

### 常见模式
- 相同的错误处理 try-catch 块
- 相同的数据转换/序列化逻辑
- 相同的 API 调用模式（URL 拼接 + header + error handling）
- 相同的 SQL 查询构建逻辑
- 相同的 UI 组件样式组合

### 检测方法
1. 提取函数体前 3 行作为签名
2. grep 相同签名出现 ≥3 次
3. 人工确认逻辑是否真正重复（不是巧合）

### 不算重复
- 只是参数不同的函数调用（如 `get_user(1)` 和 `get_user(2)`）
- 不同模块各自处理的错误类型不同

## P3: 过度设计（S2，仅标注）

### 模式
- 线性 3 步流程套了 DAG / FSM / 状态机
- 硬编码的 5 条数据用了 class + factory + strategy 模式
- 只有一个实现的接口
- 配置项只有 2 个值却用了 builder pattern
- 20 行能完成的逻辑用了 200 行（10 倍膨胀）

### 不算过度设计
- 有明确扩展计划的抽象
- 测试需要的依赖注入
- 团队约定的标准模式

## P4: God 文件/函数（S2）

### 文件级（>500 行且 ≥3 职责）
- 职责判断：文件内函数分组，每组操作不同数据/做不同事
- 常见 God 文件：`utils.py`（什么都往里塞）、`service.py`（所有业务逻辑）

### 函数级（>80 行且做 ≥3 件事）
- 做 ≥3 件事 = 调用 ≥3 类不同操作（DB 查询 + API 调用 + 文件 IO + 计算 + 渲染）
- 建议拆分点：每个"件事"提取为一个函数

## P5: 散落的映射/配置（S2）

### 模式
- 同名字典/Map 在 ≥3 个文件中独立定义
- 状态映射表（status → label）散落各处
- 平台名称/ID 映射在多处重复
- 错误码 → 错误消息映射在多处重复

### 解决方案
- 提取到 `constants.ts` / `constants.py` / 专用配置文件
- 保留原始位置的 import 指向集中定义

## P6: 类型安全缺失（S2）

### Python
- 函数参数/返回值无类型注解
- `dict` 应该用 TypedDict 或 Pydantic model
- `Any` 类型使用

### TypeScript
- `any` 类型使用
- `as` 类型断言 ≥3 次
- interface 定义在组件文件内而非独立的 `types.ts`
- `Record<string, unknown>` 而非具体类型

## P7: 错误处理缺失（S3）

### 后端
- `session.execute()` 结果未检查空值
- `one()` / `scalar_one()` 而非 `one_or_none()` / `scalar_one_or_none()`
- API 端点无 try-catch
- 外部 API 调用无超时/重试

### 前端
- `fetch` / `axios` 调用无 `.catch()`
- async 函数无 try-catch
- 错误状态不展示给用户（只有 console.error）
- Promise 无 `.catch()` 或 await 无 try-catch

## P8: 隐性依赖（S3，需谨慎）

### 模式
- 函数依赖全局状态（模块级变量、单例）
- 测试 mock 的对象和实际代码不一致
- 循环 import（A import B，B import A）
- import 时有副作用（注册路由、修改全局配置）

### 处理原则
- 只标注，不自动改
- 列出依赖链让用户判断

## 扫描排除项（不要报）

- `__init__.py` 的空文件或只有 import
- 测试文件（`test_*.py` / `*.test.ts`）
- 配置文件（`*.config.*` / `settings.py`）
- 生成代码（`node_modules` / `.next` / `dist`）
- 类型声明文件（`*.d.ts`）
- 数据库 migration 文件
- 种子数据脚本
