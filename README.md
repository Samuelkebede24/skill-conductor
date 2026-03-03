# Skill Conductor

A skill that creates, evaluates, and improves other skills. Meta-level.

Built for [OpenClaw](https://openclaw.ai/) agents running on Claude Code, but the methodology works with any LLM that supports skills/instructions.

## The Problem

Writing an agent skill looks easy. It's just a markdown file, right?

In practice, most skills fail silently. The model reads the description, ignores the body, and hallucinates the rest. You think it's working until you check the output closely.

Good skills take weeks or months of iteration. They're not documents — they're the transfer of your own expertise into a format an AI can reliably execute.

## What Skill Conductor Does

It's a cookbook for building skills, based on:

- **Anthropic's official skill-writing guide** (the full PDF methodology)
- **obra/superpowers** writing-skills best practices
- **mgechev/skills-best-practices** patterns
- **OpenClaw's built-in skill-creator**

All distilled into one skill with 5 modes:

| Mode | What it does |
|------|-------------|
| **CREATE** | Build a new skill from scratch in 7 phases |
| **EVAL** | Score any skill against 10 criteria (out of 50) |
| **EDIT** | Targeted improvements without breaking what works |
| **REVIEW** | Audit third-party skills before installing |
| **PACKAGE** | Validate and prepare for distribution |

## The TDD Approach

The key insight: **verify the agent fails without your skill first.**

1. Take a real use case
2. Run it in a clean session — no skill loaded
3. Watch the agent struggle. Document what went wrong
4. Now write the skill to fix exactly those failures
5. Run the same test again. Compare

If the agent already handles the task without the skill, you don't need the skill. Save your tokens.

## What's Inside

```
skill-conductor/
├── SKILL.md              # The main skill file (drop this into your agent)
├── references/
│   └── patterns.md       # Architecture patterns and degrees of freedom
└── scripts/
    ├── init_skill.py     # Scaffold a new skill structure
    ├── eval_skill.py     # Run evaluation against 10 criteria
    ├── package_skill.py  # Validate and package for distribution
    └── quick_validate.py # Fast frontmatter + structure check
```

## Quick Start

### For OpenClaw

Download ZIP → unpack into skills folder:

```
~/.openclaw/workspace/skills/skill-conductor/
```

Or clone:

```bash
git clone https://github.com/smixs/skill-conductor.git ~/.openclaw/workspace/skills/skill-conductor
```

### For Claude Code

Drop into your project's `.claude/skills/`:

```bash
mkdir -p .claude/skills/skill-conductor
cp SKILL.md .claude/skills/skill-conductor/
cp -r references scripts .claude/skills/skill-conductor/
```

### For other agents

The `SKILL.md` is standalone. Feed it to any LLM that accepts system instructions. The methodology (TDD, progressive disclosure, concrete templates over prose) is universal.

## Real Results

| Skill | Before | After | Iterations |
|-------|--------|-------|------------|
| OSINT research | 42/50 | 45/50 | 2 EDIT cycles |
| Autograph (vault engine) | 33/50 | 36/50 | 1 CREATE cycle |

## Key Discovery

**Never put process steps in the skill description.**

If your `description` says "Exports assets, generates specs, creates tasks" — the model follows the description and skips the body. Description = purpose + triggers only.

## License

[MIT](LICENSE)

## Author

[Serge Shima](https://shima.me) — 30+ production skills across 5 AI agents. [aimasters.me](https://aimasters.me)
