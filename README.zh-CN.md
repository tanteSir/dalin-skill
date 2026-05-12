# code-slim

[English](README.md)

**通用代码瘦身引擎，专为 AI 编程 Agent 设计。**

[code-slim](https://github.com/tanteSir/code-slim) 扫描代码库中的死代码、重复逻辑、God 文件、散落配置等问题，在独立 git 分支上执行安全的删除和重构，每一步都强制验证。

> 在 AI 辅助开发的时代，代码膨胀的速度比以往更快。AI 生成代码很快，但清理代码很慢。code-slim 是一个给 AI coding agent 用的 skill，让 agent 能系统地扫描、诊断、安全地精简代码库。

## 为什么需要

| 问题 | code-slim 的解法 |
|------|-----------------|
| AI 生成了死代码但从不清理 | 3 层引用验证确认零引用后才标记为 S0 可删 |
| 重复代码越积越多 | ≥3 次相似自动识别，提取共享函数 |
| God 文件越来越大 | 量化阈值（500行/3职责）自动标记，输出拆分建议 |
| 重构怕改坏 | 独立 `*-slim` 分支 + 每条改动独立验证 + 4 级安全分级 |
| 每次"瘦身"标准不一样 | references 固定扫描模式、白名单、量化阈值 |

## 工作流程

```
阶段 0: 创建 *-slim 分支（隔离所有改动）
    ↓
阶段 1: 安全扫描（grep 3 层验证，确认安全等级 S0-S3）
    ↓
阶段 2: 按 8 条规则扫描（R1-R8，量化阈值判据）
    ↓
阶段 3: 输出优化清单（表格，按优先级排序）
    ↓
阶段 4: 逐条执行 + 每条验证（import / build / test）
    ↓
阶段 5: 更新架构路由表
    ↓
用户确认 → 合并分支 or 回滚
```

## 8 条扫描规则

| 规则 | 扫描内容 | 安全等级 |
|------|---------|---------|
| **R1** 死代码 | 零引用的文件/函数/变量/类 | S0 — 直接删 |
| **R2** 重复代码 | ≥3 次相似代码，≥5 行 | S1 — 提取共享函数 |
| **R3** 过度设计 | 20 行逻辑用了 200 行 | 标注，不强制 |
| **R4** God 文件/函数 | >500行且≥3职责 / >80行且做≥3事 | S2 — 建议拆分 |
| **R5** 散落配置 | 同名字典在≥3个文件独立定义 | S2 — 集中管理 |
| **R6** 类型安全缺失 | `any` / 缺类型注解 | S2 |
| **R7** 错误处理缺失 | DB 查询无空值检查、API 无 try-catch | S3 |
| **R8** Token 浪费估算 | 汇总浪费行数和估算 token 数 | 报告 |

## 安全机制

### 4 级安全分级

| 等级 | 含义 | 操作权限 | 必须验证 |
|------|------|---------|---------|
| **S0** | 零引用的死代码 | 直接删除 | grep 3 层零命中 |
| **S1** | 重复代码提取 | 提取为共享函数，调用方改 import | 提取后所有调用方 import 正常 |
| **S2** | 内部重构 | 拆分文件/外置数据/参数化 | 不改外部接口，行为等价 |
| **S3** | 可能影响行为 | 需跑完整测试 | 必须有测试覆盖 |

### 3 层死代码验证

标记任何代码为 S0（可安全删除）前，3 层验证必须全部零命中：

```
第 1 层: grep import 语句        → 文件名是否被 import？
第 2 层: grep 导出符号           → 类名/函数名是否被引用？
第 3 层: grep barrel export      → 是否通过 __init__.py / index.ts 重新导出？
```

任一层有命中 → 不是死代码，降级为 S2 或跳过。

### 项目级白名单

框架入口、ORM Model、Celery 任务等隐性引用的代码默认受保护。详见 [references/project-rules.md](references/project-rules.md)。

## 文件结构

```
code-slim/
├── SKILL.md                              # 主提示词 — 5 阶段完整流程
└── references/
    ├── scan-patterns.md                  # 扫描模式清单 (P1-P8)
    ├── project-rules.md                  # 项目白名单 (不能删的文件/模块)
    └── metrics-threshold.md              # 量化阈值 (什么算"胖")
```

### 为什么需要 references？

| 文件 | 解决的问题 |
|------|-----------|
| `scan-patterns.md` | 每次扫描标准一致，不靠 LLM 自己想该查什么 |
| `project-rules.md` | 防止误删框架入口、ORM Model、Celery 任务等隐性引用 |
| `metrics-threshold.md` | "500行算大文件"等标准固定，不同批次扫描结果可比 |

## 安装

### Claude Code

```bash
# 方式 1: 放到全局 skills 目录
cp -r code-slim/ ~/.claude/skills/

# 方式 2: 放到 agents skills 目录
cp -r code-slim/ ~/.agents/skills/
```

### OpenCode

```bash
cp -r code-slim/ ~/.config/opencode/skills/
```

### OpenAI Codex / 其他 Agent

将 `SKILL.md` 内容放入项目的 `AGENTS.md` 或 agent 的指令文件中。

## 使用方法

在 AI coding agent 对话中说：

- "代码瘦身" / "瘦身扫描" / "slim" / "精简代码"

Agent 会自动：

1. 创建 `{当前分支}-slim` 分支
2. 扫描整个代码库，输出优化清单
3. 等你确认后逐条执行
4. 每条改动验证通过后才继续下一条
5. 全部完成后等你确认是否合并

## 语言支持

| 语言 | 死代码检测 | 重复代码 | 重构 | 验证方式 |
|------|-----------|---------|------|---------|
| **Python** | `__init__.py` import 算引用 | ✓ | ✓ | `pytest` / `python -c "import ..."` |
| **TypeScript/React** | barrel export 算引用 | ✓ | ✓ | `npm run build` |
| **Go** | 编译器警告覆盖 | 接口抽取 | ✓ | `go build` |
| **Java** | 编译器警告覆盖 | 泛型化 | ✓ | `mvn compile` |
| **Rust** | 编译器警告覆盖 | 泛型化 | ✓ | `cargo build` |

## 扫描输出示例

```markdown
| # | 安全等级 | 规则 | 文件 | 行数 | 问题 | 建议 | 预估削减 |
|---|---------|------|------|------|------|------|---------|
| 1 | S0 | R1 | utils/old_helpers.py | 45 | 零引用 | 删除 | 45 |
| 2 | S0 | R1 | constants.ts:12-18 | 6 | 未使用的 STATUS_MAP | 删除 | 6 |
| 3 | S1 | R2 | service.py:80-95 (x3) | 48 | 重复错误处理 | 提取 handle_api_error() | 32 |
| 4 | S2 | R4 | articles/page.tsx | 520 | God 组件 (4个职责) | 拆分为4个组件 | - |
| 5 | S2 | R5 | 3 个文件 | 21 | 平台名映射散落 | 集中到 constants.ts | 14 |
|    |    |    |    | **合计** | | | **~97 行** |
```

## 相关 Skills

- **[neat-freak](https://github.com/...)** — 文档和记忆的洁癖级同步（code-slim 瘦代码，neat-freak 瘦文档）

## 联系方式

有想法或建议？找我聊：

- **微信**: `tanteSir`
- **GitHub Issues**: [提交 Issue](https://github.com/tanteSir/code-slim/issues)

## License

MIT
