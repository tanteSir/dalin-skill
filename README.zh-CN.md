# 🧰 Dalin Skills

[English](README.md)

**我自己每天在用的 AI Skills，开源在这里。**

都是在自己项目里跑通了一段时间，确实省事，才搬出来开源的。没什么花活，就是几个实用的东西。

- **Skills** — Agent 能直接加载的结构化指令集，遵循 [Agent Skills](https://agentskills.io) 开放标准。Claude Code、Codex、OpenCode、OpenClaw 都能装

![License](https://img.shields.io/badge/License-MIT-3B82F6?style=for-the-badge) ![Skills](https://img.shields.io/badge/Skills-1-10B981?style=for-the-badge)

![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-D97706?style=flat-square&logo=anthropic&logoColor=white) ![Codex](https://img.shields.io/badge/Codex-Skill-10B981?style=flat-square&logo=openai&logoColor=white) ![OpenCode](https://img.shields.io/badge/OpenCode-Skill-3B82F6?style=flat-square) ![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-8B5CF6?style=flat-square)

---

## Skills

| Skill | 一句话 | 详情 |
|-------|--------|------|
| 🔪 [**code-slim（代码瘦身）**](./code-slim/README.zh-CN.md) | 扫描死代码、重复逻辑、God 文件，在独立分支上安全精简代码库 | [SKILL.md](./code-slim/SKILL.md) · [中文说明](./code-slim/README.zh-CN.md) |

---

## 安装

在 Claude Code、Codex、OpenClaw 等支持 Skill 的 Agent 里，直接说：

```
帮我安装这个 skill：https://github.com/tanteSir/dalin-skill/tree/main/<skill-name>
```

把 `<skill-name>` 换成你想装的那个，比如 `code-slim`。Agent 会自己 clone 到对应目录。

或者手动安装：

```bash
# Claude Code
cp -r <skill-name>/ ~/.claude/skills/

# OpenCode
cp -r <skill-name>/ ~/.config/opencode/skills/

# 通用
cp -r <skill-name>/ ~/.agents/skills/
```

---

## 联系方式

有想法或建议？找我聊：

- **微信**: `tanteSir`
- **GitHub Issues**: [提交 Issue](https://github.com/tanteSir/dalin-skill/issues)

## License

[MIT](./LICENSE) · 自由使用 / 修改 / 再分发

Made by [@tanteSir](https://github.com/tanteSir)
