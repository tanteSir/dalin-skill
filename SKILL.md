# Skill: code-slim — 通用代码瘦身引擎

> 语言无关的代码瘦身工具。扫描 → 诊断 → 执行 → 验证。
> 触发词：代码瘦身、瘦身扫描、slim、精简代码

## 核心原则

1. **在独立分支操作** — 所有瘦身改动在 `{当前分支}-slim` 分支进行，不污染主线
2. **只做静态分析确定的改动** — 不猜测运行时行为
3. **每条改动独立可回滚** — 不批量执行
4. **改完必须验证** — 后端 `pytest` / 前端 `npm run build`，无测试则 `import` 验证
5. **不改运行时语义** — 提取/删除后行为必须一致

---

## 第零阶段：创建瘦身分支（最先执行）

瘦身开始前，**必须先创建独立分支**，将所有改动隔离：

```bash
# 获取当前分支名
current_branch=$(git branch --show-current)

# 创建瘦身分支
git checkout -b "${current_branch}-slim"
```

**规则：**
- 分支名格式：`{当前分支}-slim`（如 `main-slim`、`feature/auth-slim`）
- 所有瘦身改动（删除、提取、拆分、文档更新）都在此分支进行
- **绝不自动合并回原分支** — 等用户确认完成后手动合并或说"合并回去"
- 如果分支已存在，直接 checkout 到该分支继续

**分支生命周期：**
```
创建分支 → 扫描 → 执行 → 验证 → 用户确认 → 用户自行合并/删除分支
```

---

## 第一阶段：安全扫描

> **扫描前必读：** [references/scan-patterns.md](references/scan-patterns.md)（扫描模式清单）、[references/project-rules.md](references/project-rules.md)（白名单，不能删的文件/模块）

对每个疑似问题，**用 grep 验证引用关系**，确认安全等级后才可列入清单。

### 安全等级

| 等级 | 含义 | 操作权限 | 必须验证 |
|------|------|---------|---------|
| S0 | 零引用的死代码 | 直接删除 | grep 零命中 |
| S1 | 重复代码提取 | 提取为共享函数，调用方改 import | 提取后所有调用方 import 正常 |
| S2 | 内部重构 | 拆分文件/外置数据/参数化 | 不改外部接口，行为等价 |
| S3 | 可能影响行为 | 需跑完整测试 | 必须有测试覆盖 |

### 验证命令模板

**S0 判定必须通过全部 3 层验证（缺一不可）：**

```bash
# 第 1 层：文件名/模块名是否被 import
grep -rn "import.*filename\|from.*filename" --include="*.py" --include="*.ts" --include="*.tsx" | grep -v "filename.py\|filename.ts"

# 第 2 层：导出的类名/函数名是否被引用（最关键！很多模块通过类名间接引用）
# 先读取文件，提取所有 class 和 def/async def 名称，然后逐个 grep
grep -rn "ClassName\|exported_function" --include="*.py" --include="*.ts" --include="*.tsx" | grep -v "filename.py\|filename.ts\|class ClassName\|def function_name"

# 第 3 层：__init__.py 或 barrel export 是否 re-export 了该模块
grep -rn "filename" --include="__init__.py" --include="index.ts" */*/
```

**判定规则：**
- 3 层全部零命中 → **S0 可删**
- 任一层有命中 → **不是死代码**，降级为 S2 或跳过
- 第 2 层尤其重要：Python 常见 `from xxx.service import YyyClass`，文件名搜不到但类名能搜到

# 变量/常量级
grep -rn "CONSTANT_NAME" --include="*.py" --include="*.ts" --include="*.tsx" | grep -v "CONSTANT_NAME =\|const CONSTANT_NAME"
```

---

## 第二阶段：按规则扫描

> 量化标准见 [references/metrics-threshold.md](references/metrics-threshold.md)，扫描时以此为判据。

### R1: 死代码（零引用）

- 文件/函数/变量/常量/类 — grep 确认零引用后标记 S0
- 前端额外检查：未使用的 state、未使用的 import

### R2: 重复代码（≥3 次相同/高度相似，≥5 行）

- 输出所有出现位置和行号
- **额外检查：不对称代码** — 如果 A 处存了 X 字段，B 处读 X 字段但 A 从没写，这是 bug 而非重复

### R3: 过度设计

- 线性流程套了 DAG/FSM；硬编码数据用了 class 包装；20 行能搞定用了 200 行
- 只标注，不强制删

### R4: God 文件/函数

- **文件**：>500 行且 ≥3 个独立职责
- **函数/组件**：>80 行且做 ≥3 件事
- 输出职责清单和建议拆分点

### R5: 散落的映射/配置

- 同名/同义字典在 ≥3 个文件中独立定义 → 集中管理

### R6: 类型安全缺失

- `any` / `Record<string, unknown>` 手动 cast ≥3 次 / interface 散落

### R7: 错误处理缺失（新增）

- DB 查询用 `one()` / `scalar_one()` 而非 `one_or_none()` / `scalar_one_or_none()` — 查不到直接 500
- API 端点无 try-catch，异常直接穿透
- 前端 async 操作无 catch，错误不展示给用户

### R8: Token 浪费估算

- 汇总：总浪费行数、估算 token 数

---

## 第三阶段：输出优化清单

```markdown
| # | 安全等级 | 规则 | 文件 | 行数 | 问题 | 建议 | 预估削减 |
|---|---------|------|------|------|------|------|---------|
```

优先级：S0 > S1 > S2 > S3，同级按行数降序。

---

## 第四阶段：执行（逐条 + 验证）

用户确认后逐条执行。**每条必须走完整流程：**

### 4.1 通用执行步骤

1. 执行改动（删除/提取/拆分）
2. **Import 验证**：所有修改过的文件必须 import 成功
3. **构建验证**：`pytest` / `npm run build` / 等价命令
4. 更新清单状态

### 4.2 组件/模块拆分检查清单（关键！）

拆分 God 文件时，**必须逐项检查**：

- [ ] **Props/参数完整性**：拆出的组件接收了所有必要的 props？特别是回调函数（onXxx）、状态设置器（setXxx）
- [ ] **State 归属正确**：哪个组件拥有 state、哪个组件只读？拆分后 state 在正确的位置？
- [ ] **回调链完整**：子组件的回调 → 父组件的处理 → API 调用 → 状态更新，整条链通？
- [ ] **条件渲染路径**：switch/case 或 if/else 的每个分支都能正确渲染？
- [ ] **重复 UI 元素**：拆分后是否有两个按钮/面板做同一件事？合并或去掉多余的
- [ ] **死组件**：拆出后没被任何地方引用的组件？直接删

### 4.3 状态流审计（关键！）

改动涉及**异步状态**（loading/submitting/running）时，必须追踪：

1. **状态谁设的**：`setSubmitting(true)` 在哪？有几个入口？
2. **状态谁清的**：`setSubmitting(false)` 在哪？每个设 true 的路径都有对应的清 false 吗？
3. **自动触发逻辑**：有没有 useEffect/定时器/事件监听器会自动触发操作？触发条件对吗？
4. **乐观更新链**：乐观更新 → SSE 确认 → 清理 optimistic ref → 清 submitting，每个环节都通？
5. **每个分支走一遍**：在脑中模拟 status A → B → C 的每个转换，submitting 在每步的值是什么？

**模板：画出状态流转图**
```
[状态A] --用户操作--> setX(true) --> [状态B] --SSE确认--> setX(false) --> [状态C]
[状态A] --自动触发--> setX(true) --> [状态B] --???--> setX(false) ??? ← 检查这里
```

### 4.4 错误处理加固（S2/S3 必须）

- DB 查询：`scalar_one()` → `scalar_one_or_none()` + 404
- API 端点：确保有异常处理，不会把 500 暴露给用户
- 前端操作：async 操作有 catch + 用户可见的错误提示

---

## 第五阶段：更新架构路由表

生成或更新 `docs/ARCHITECTURE-ROUTE.md`。

**原则：**
- 每个功能一行，不超过 4 列（功能 / 文件 / 关键函数 / 备注）
- AI 读了路由表能在 1-2 次 grep 内定位代码
- 表格总行数 ≤ 40 行
- **拆分后的组件/模块必须列入**，标注拆分关系

---

## 语言适配提示

### Python (FastAPI / Django / Flask)
- 死代码：`__init__.py` 导入也算引用
- 提取：注意 `flag_modified` / ORM 状态
- 验证：`python -c "from app.xxx import Yyy; print('OK')"` 无 DB 也能验证 import
- 错误处理：`scalar_one_or_none()` + `raise HTTPException(404)`

### TypeScript/React (Next.js / React)
- 死代码：注意 barrel export（index.ts re-export）
- 拆分：走 4.2 检查清单，特别关注 props 完整性和回调链
- 验证：`npm run build` 必须通过，TypeScript 编译无错误
- 状态流：走 4.3 审计，特别关注 loading/submitting 状态

### Go / Java / Rust
- 死代码：编译器警告已覆盖大部分
- 重复代码：接口抽取、泛型化
- 验证：`go build` / `mvn compile` / `cargo build`

---

## 参考资料

- **[references/scan-patterns.md](references/scan-patterns.md)** — 完整的扫描模式清单（P1-P8），扫描时逐项检查
- **[references/project-rules.md](references/project-rules.md)** — 项目级白名单，哪些代码不能删
- **[references/metrics-threshold.md](references/metrics-threshold.md)** — 量化阈值，什么算"胖"

---

## 执行纪律

1. **分支隔离** — 所有改动在 `*-slim` 分支，绝不自动合并
2. **不猜需求** — 不确定就问用户
3. **不过度设计** — 能跑就行，先通再优
4. **不自动提交** — 每条改动验证通过后等用户说"提交"才 commit
5. **改完必须验证** — 没有例外
6. **记录每个坑** — 执行中遇到的意外问题写入 SESSION-CONTEXT 或 AGENTS.md

---

## 收尾：用户确认后

所有条目执行完毕且验证通过后：

1. **提示用户当前状态**：`*-slim` 分支上所有改动已就绪
2. **等待用户指令**：
   - 用户说"合并回去" / "瘦身完成" → 合并分支并删除 slim 分支
   - 用户说"先看看" → 停下来，等用户自行 review
   - 用户说"回滚" → `git checkout {原分支}` 放弃改动
3. **合并操作**：
   ```bash
   git checkout {原分支}
   git merge {原分支}-slim
   git branch -d {原分支}-slim
   ```
