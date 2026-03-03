# Skill Conductor

A skill that creates, evaluates, and improves other agent skills.

Synthesized from four sources:

1. **[Anthropic Skill Creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)** — base framework, scaffold scripts, validation, packaging
2. **[The Complete Guide to Building Skills for Claude](https://claude.com/blog/complete-guide-to-building-skills-for-claude)** ([PDF](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf?hsLang=en)) — 32 pages of best practices, 5 architecture patterns (sequential workflow, iterative refinement, context-aware selection, domain intelligence, multi-MCP coordination), success metrics (90% triggering rate, 0 failed calls), three skill categories
3. **[Superpowers / writing-skills](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md)** by [Jesse Vincent](https://github.com/obra/superpowers) — TDD approach to skills, the "description trap" discovery
4. **[Skills Best Practices](https://github.com/mgechev/skills-best-practices)** by [Minko Gechev](https://blog.mgechev.com/2026/02/26/skill-eval/) (Angular team lead, Google) — three-stage LLM validation, eval methodology

## What all four share

- Progressive disclosure (three loading levels: metadata → body → references)
- No junk files inside the skill (no README, no CHANGELOG)
- Description = triggers, not process
- Scripts for fragile operations, text for judgment calls
- kebab-case, third person, imperative voice

## What Skill Conductor adds

- Built-in eval system with automatic scoring across 5 axes (10 criteria, max 50 points)
- "Process leak" detector — catches workflow in description automatically
- Problem → Signal → Fix table for common failure modes
- Degrees of freedom: bridge (strict rules) vs. field (route freedom)

## 5 Modes

| Mode | What it does |
|------|-------------|
| **CREATE** | Build a new skill from scratch in 7 phases (with TDD) |
| **EVAL** | Score any skill against 10 criteria, max 50 points |
| **EDIT** | Targeted improvements without breaking what works |
| **REVIEW** | Audit third-party skills before installing them |
| **PACKAGE** | Validate and prepare for distribution |

## Installation

Download ZIP → unpack so the structure is:

```
skills/
└── skill-conductor/
    ├── SKILL.md
    ├── references/
    │   └── patterns.md
    └── scripts/
        ├── init_skill.py
        ├── eval_skill.py
        ├── package_skill.py
        └── quick_validate.py
```

**OpenClaw:** drop `skills/skill-conductor/` into `~/.openclaw/workspace/skills/`

**Claude Code:** drop into `.claude/skills/`

The skill auto-activates when the agent detects a skill-building task.

## Key discovery

Never put process steps in the skill description. If your description says "exports assets, generates specs, creates tasks" — the model follows the description and skips the body. Tested experimentally.

```yaml
# ✅ Good
description: Analyze design files for developer handoff. Use when user uploads .fig files.

# ❌ Bad — model follows this and ignores SKILL.md body
description: Exports Figma assets, generates specs, creates Linear tasks, posts to Slack.
```

## License

MIT
