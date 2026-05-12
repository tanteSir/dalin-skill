# code-slim

**Language-agnostic code slimming engine for AI coding agents.** 通用代码瘦身引擎。

[code-slim](#) scans your codebase for dead code, duplication, god files, scattered configs, and more — then executes safe deletions and refactors on an isolated git branch with mandatory verification at every step.

> **在 AI 辅助开发的时代，代码膨胀的速度比以往更快。** AI 生成代码很快，但清理代码很慢。code-slim 是一个给 AI coding agent 用的 skill，让 agent 能系统地扫描、诊断、安全地精简代码库。

## Why / 为什么需要

| Problem | code-slim 的解法 |
|---------|-----------------|
| AI 生成了死代码但从不清理 | 3 层引用验证确认零引用后才标记为 S0 可删 |
| 重复代码越积越多 | ≥3 次相似自动识别，提取共享函数 |
| God 文件越来越大 | 量化阈值（500行/3职责）自动标记，输出拆分建议 |
| 重构怕改坏 | 独立 `*-slim` 分支 + 每条改动独立验证 + 安全等级分级 |
| 每次"瘦身"标准不一样 | references 固定扫描模式、白名单、量化阈值 |

## How It Works / 工作流程

```
Stage 0: 创建 *-slim 分支（隔离所有改动）
    ↓
Stage 1: 安全扫描（grep 3 层验证，确认安全等级 S0-S3）
    ↓
Stage 2: 按 8 条规则扫描（R1-R8，量化阈值判据）
    ↓
Stage 3: 输出优化清单（表格，按优先级排序）
    ↓
Stage 4: 逐条执行 + 每条验证（import / build / test）
    ↓
Stage 5: 更新架构路由表
    ↓
用户确认 → 合并分支 or 回滚
```

## 8 Scan Rules / 8 条扫描规则

| Rule | What it finds | Safety |
|------|--------------|--------|
| **R1** Dead code | 零引用的文件/函数/变量/类 | S0 — 直接删 |
| **R2** Duplication | ≥3 次相似代码，≥5 行 | S1 — 提取共享函数 |
| **R3** Over-engineering | 20 行逻辑用了 200 行 | 标注，不强制 |
| **R4** God file/function | >500行且≥3职责 / >80行且做≥3事 | S2 — 建议拆分 |
| **R5** Scattered configs | 同名字典在≥3个文件独立定义 | S2 — 集中管理 |
| **R6** Type safety gaps | `any` / 缺类型注解 | S2 |
| **R7** Error handling gaps | DB 查询无空值检查、API 无 try-catch | S3 |
| **R8** Token waste estimate | 汇总浪费行数和估算 token 数 | 报告 |

## Safety System / 安全机制

### 4-Level Safety Classification

| Level | Meaning | Action | Required Verification |
|-------|---------|--------|-----------------------|
| **S0** | Zero-reference dead code | Direct delete | grep 3-layer zero hits |
| **S1** | Duplicate code extraction | Extract shared function | All callers import OK |
| **S2** | Internal refactor | Split / externalize / parameterize | No external API change |
| **S3** | May affect behavior | Full test run required | Test coverage required |

### 3-Layer Dead Code Verification

Before marking anything as S0 (safe to delete), all 3 layers must return zero hits:

```
Layer 1: grep import statements       → is the filename imported anywhere?
Layer 2: grep exported symbols        → are class/function names referenced?
Layer 3: grep barrel exports          → is it re-exported via __init__.py / index.ts?
```

Any hit → not dead code, downgrade to S2 or skip.

### Project-Level Whitelist

Framework entry points, ORM models, Celery tasks, and other implicitly-referenced code are protected by default. See [references/project-rules.md](references/project-rules.md).

## File Structure / 文件结构

```
code-slim/
├── SKILL.md                              # 主提示词 — 5 阶段完整流程
└── references/
    ├── scan-patterns.md                  # 扫描模式清单 (P1-P8)
    ├── project-rules.md                  # 项目白名单 (不能删的文件/模块)
    └── metrics-threshold.md              # 量化阈值 (什么算"胖")
```

### Why references?

| 文件 | 解决的问题 |
|------|-----------|
| `scan-patterns.md` | 每次扫描标准一致，不靠 LLM 自己想该查什么 |
| `project-rules.md` | 防止误删框架入口、ORM Model、Celery 任务等隐性引用 |
| `metrics-threshold.md` | "500行算大文件"等标准固定，不同批次扫描结果可比 |

## Install / 安装

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

## Usage / 使用

在 AI coding agent 对话中说：

- "代码瘦身" / "瘦身扫描" / "slim" / "精简代码"

Agent 会自动：

1. 创建 `{当前分支}-slim` 分支
2. 扫描整个代码库，输出优化清单
3. 等你确认后逐条执行
4. 每条改动验证通过后才继续下一条
5. 全部完成后等你确认是否合并

## Language Support / 语言支持

| Language | Dead Code | Duplication | Refactor | Verification |
|----------|-----------|-------------|----------|-------------|
| **Python** | `__init__.py` import 算引用 | ✓ | ✓ | `pytest` / `python -c "import ..."` |
| **TypeScript/React** | barrel export 算引用 | ✓ | ✓ | `npm run build` |
| **Go** | 编译器警告覆盖 | 接口抽取 | ✓ | `go build` |
| **Java** | 编译器警告覆盖 | 泛型化 | ✓ | `mvn compile` |
| **Rust** | 编译器警告覆盖 | 泛型化 | ✓ | `cargo build` |

## Example Output / 扫描输出示例

```markdown
| # | Safety | Rule | File | Lines | Issue | Suggestion | Est. Reduction |
|---|--------|------|------|-------|-------|------------|---------------|
| 1 | S0 | R1 | utils/old_helpers.py | 45 | Zero references | Delete | 45 |
| 2 | S0 | R1 | constants.ts:12-18 | 6 | Unused STATUS_MAP | Delete | 6 |
| 3 | S1 | R2 | service.py:80-95 (x3) | 48 | Duplicate error handling | Extract handle_api_error() | 32 |
| 4 | S2 | R4 | articles/page.tsx | 520 | God component (4 responsibilities) | Split into 4 components | - |
| 5 | S2 | R5 | 3 files | 21 | Platform name map scattered | Consolidate to constants.ts | 14 |
|    |    |    |    | **Total** | | | **~97 lines** |
```

## Related Skills / 相关 Skills

- **[neat-freak](https://github.com/...)** — 文档和记忆的洁癖级同步（code-slim 瘦代码，neat-freak 瘦文档）

## Contact / 联系

有想法或建议？找我聊：

- **WeChat / 微信**: `tanteSir`
- **GitHub Issues**: [提交 Issue](https://github.com/tanteSir/code-slim/issues)

## License

MIT
