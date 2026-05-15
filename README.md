# code-slim

[中文](README.zh-CN.md)

**Language-agnostic code slimming engine for AI coding agents.**

[code-slim](https://github.com/tanteSir/code-slim) scans your codebase for dead code, duplication, god files, scattered configs, and more — then executes safe deletions and refactors on an isolated git branch with mandatory verification at every step.

> In the era of AI-assisted development, code bloat happens faster than ever. AI generates code quickly, but cleaning it up is slow. code-slim is an AI coding agent skill that systematically scans, diagnoses, and safely trims your codebase.

## Why

| Problem | How code-slim solves it |
|---------|------------------------|
| AI generates dead code but never cleans it | 3-layer reference verification confirms zero usage before marking as S0 (safe to delete) |
| Duplicate code accumulates | Auto-detects ≥3 similar occurrences, extracts shared functions |
| God files keep growing | Quantified thresholds (500 lines / 3 responsibilities) auto-flag, outputs split suggestions |
| Fear of breaking things during refactor | Isolated `*-slim` branch + per-item independent verification + 4-level safety classification |
| Inconsistent "slimming" standards across runs | references/ fixes scan patterns, whitelists, and quantified thresholds |

## How It Works

```
Stage 0: Create *-slim branch (isolate all changes)
    ↓
Stage 1: Safety scan (grep 3-layer verification, classify S0-S3)
    ↓
Stage 2: Scan by 8 rules (R1-R8, quantified thresholds)
    ↓
Stage 3: Output optimization checklist (sorted by priority)
    ↓
Stage 4: Execute one by one + verify each (import / build / test)
    ↓
Stage 5: Update architecture route table
    ↓
User confirms → merge branch or rollback
```

## 8 Scan Rules

| Rule | What it finds | Safety |
|------|--------------|--------|
| **R1** Dead code | Zero-reference files/functions/variables/classes | S0 — direct delete |
| **R2** Duplication | ≥3 similar occurrences, ≥5 lines | S1 — extract shared function |
| **R3** Over-engineering | 20-line logic wrapped in 200 lines | Flag only, no force |
| **R4** God file/function | >500 lines & ≥3 responsibilities / >80 lines & ≥3 tasks | S2 — suggest split |
| **R5** Scattered configs | Same-name dicts defined independently in ≥3 files | S2 — consolidate |
| **R6** Type safety gaps | `any` / missing type annotations | S2 |
| **R7** Error handling gaps | DB queries without null checks, API without try-catch | S3 |
| **R8** Token waste estimate | Total wasted lines and estimated token count | Report |

## Safety System

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

## File Structure

```
code-slim/
├── SKILL.md                              # Main prompt — 5-stage workflow
└── references/
    ├── scan-patterns.md                  # Scan pattern checklist (P1-P8)
    ├── project-rules.md                  # Project whitelist (protected files/modules)
    └── metrics-threshold.md              # Quantified thresholds (what counts as "fat")
```

### Why references?

| File | What it solves |
|------|---------------|
| `scan-patterns.md` | Consistent scan standards across runs — no relying on LLM to invent patterns |
| `project-rules.md` | Prevents accidental deletion of framework entries, ORM models, Celery tasks, etc. |
| `metrics-threshold.md` | Fixed standards like "500 lines = large file" — comparable results across batches |

## Install

### Claude Code

```bash
# Option 1: Global skills directory
cp -r code-slim/ ~/.claude/skills/

# Option 2: Agents skills directory
cp -r code-slim/ ~/.agents/skills/
```

### OpenCode

```bash
cp -r code-slim/ ~/.config/opencode/skills/
```

### OpenAI Codex / Other Agents

Add `SKILL.md` content to your project's `AGENTS.md` or agent instruction file.

## Usage

In your AI coding agent conversation, say:

- "代码瘦身" / "瘦身扫描" / "slim" / "精简代码"

The agent will automatically:

1. Create a `{branch}-slim` branch
2. Scan the entire codebase and output an optimization checklist
3. Wait for your confirmation before executing
4. Verify each change before proceeding to the next
5. Wait for your confirmation to merge when all done

## Language Support

| Language | Dead Code | Duplication | Refactor | Verification |
|----------|-----------|-------------|----------|-------------|
| **Python** | `__init__.py` imports count | ✓ | ✓ | `pytest` / `python -c "import ..."` |
| **TypeScript/React** | Barrel exports count | ✓ | ✓ | `npm run build` |
| **Go** | Compiler warnings cover most | Interface extraction | ✓ | `go build` |
| **Java** | Compiler warnings cover most | Generics | ✓ | `mvn compile` |
| **Rust** | Compiler warnings cover most | Generics | ✓ | `cargo build` |

## Example Output

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

## Related Skills

- **[neat-freak](https://github.com/...)** — OCD-level doc & memory sync (code-slim trims code, neat-freak trims docs)

## Contact

Ideas or suggestions? Let's talk:

- **WeChat**: `tanteSir`
- **GitHub Issues**: [Submit Issue](https://github.com/tanteSir/code-slim/issues)

## License

MIT
