---
name: skill-conductor
description: Create, edit, evaluate, and package agent skills. Use when building a new skill from scratch, improving an existing skill, running TDD-style evaluation on a skill, reviewing third-party skills for quality, or packaging skills for distribution. Not for using skills or general coding tasks.
---

# Skill Conductor

Full lifecycle management for agent skills: design → build → test → evaluate → package.

Synthesized from Anthropic official guide, obra/superpowers writing-skills, mgechev/skills-best-practices, and OpenClaw skill-creator.

## Modes

Detect mode from context. If ambiguous, ask.

1. **CREATE** - new skill from scratch
2. **EDIT** - improve existing skill
3. **EVAL** - TDD-style evaluation of a skill
4. **REVIEW** - assess third-party skill quality
5. **PACKAGE** - validate and package for distribution

---

## Mode 1: CREATE

### Phase 1: Use Cases (do not skip)

Extract 2-3 concrete scenarios before writing anything.

Ask:
- "What specific task should this skill handle?"
- "What would a user say to trigger it?"
- "What should NOT trigger it?"

Conclude when: clear picture of what the skill does, for whom, and when.

### Phase 2: Baseline (TDD RED)

Before writing the skill, verify the agent fails without it:

1. Take one use case from Phase 1
2. Run it in a clean session without the skill
3. Document: what went wrong, what the agent guessed, what it missed
4. This is the baseline. If the agent already handles it perfectly, the skill is unnecessary

Do not skip baseline. If you didn't watch the agent fail, you don't know what to teach.

### Phase 3: Architecture

Choose primary pattern (can combine):

| Pattern | Use when |
|---|---|
| Sequential workflow | clear step-by-step process |
| Iterative refinement | output improves with cycles |
| Context-aware selection | same goal, different tools by context |
| Domain intelligence | specialized knowledge beyond tool access |
| Multi-MCP coordination | workflow spans multiple services |

Choose degrees of freedom:

| Freedom | When | Example |
|---|---|---|
| Low (scripts) | fragile, error-prone, must be exact | PDF rotation, API calls |
| Medium (pseudocode) | preferred pattern exists, some variation ok | data processing |
| High (text) | multiple approaches valid, judgment needed | design decisions |

### Phase 4: Scaffold

Initialize skill structure:

```bash
python scripts/init_skill.py <skill-name> --path <output-dir> [--resources scripts,references,assets]
```

Or create manually:
```
skill-name/
├── SKILL.md          # required
├── scripts/          # deterministic operations
├── references/       # detailed docs, loaded on demand
└── assets/           # templates, images for output
```

### Phase 5: Write SKILL.md

#### Frontmatter

```yaml
---
name: kebab-case-name
description: [purpose in one sentence]. Use when [triggers]. Do NOT use for [negative triggers].
---
```

Rules (non-negotiable):
- `name`: lowercase, digits, hyphens only. no consecutive hyphens. matches folder name. max 64 chars
- `description`: max 1024 chars. no angle brackets (< >). no "claude" or "anthropic" in name
- description = purpose + triggers. **NEVER workflow/process steps** (tested: agent follows description instead of reading body)
- start triggers with "Use when..."
- include negative triggers ("Do NOT use for...")
- third person, imperative voice

```yaml
# ✅ GOOD: purpose + triggers, no process
description: Analyze Figma design files for developer handoff. Use when user uploads .fig files or asks for "design specs". Do NOT use for Sketch or Adobe XD files.

# ❌ BAD: process in description (agent will skip body)
description: Exports Figma assets, generates specs, creates Linear tasks, posts to Slack.

# ❌ BAD: too vague
description: Helps with designs.
```

#### Body structure

```markdown
# Skill Name

## Overview
What this enables. 1-2 sentences. Core principle.

## [Main sections - workflow steps, task categories, or capabilities]
Step-by-step with numbered sequences.
Concrete templates over prose.
Imperative voice: "Extract the text..." not "You should extract..."

## Common Mistakes
What goes wrong + how to fix.

## Troubleshooting (if applicable)
Error: [message] → Cause: [why] → Fix: [how]
```

#### Writing rules

- **Imperative, third person.** "Extract the data" not "I will extract" or "You should extract"
- **One term per concept.** Pick "template" and stick with it. Not template/boilerplate/scaffold
- **Step-by-step numbering.** Decision trees mapped explicitly: "Step 2: If X, do A. Otherwise skip to Step 3"
- **Concrete templates > prose.** Pattern-matching beats paragraphs
- **Progressive disclosure.** SKILL.md = brain (<500 lines). References = details (loaded on demand). One level deep only
- **No junk files.** No README, CHANGELOG, INSTALLATION_GUIDE inside the skill
- **Token budget.** Frequently loaded: <200 words. Standard: <500 words. Heavy reference: move to references/

#### Scripts

Bundle when:
- same code rewritten repeatedly
- operation is fragile, variation = bug
- deterministic reliability needed

Scripts must:
- return descriptive stdout/stderr on failure
- be tested by running before packaging
- be tiny CLIs, not library code

### Phase 6: Verify (TDD GREEN)

Run the same use case from Phase 2, now with the skill active.

1. Does the skill trigger automatically?
2. Does the agent follow instructions from body (not just description)?
3. Does the output meet the use case requirements?
4. Does it NOT trigger on unrelated queries?

If any fail → back to Phase 5.

### Phase 7: Refactor

1. Find new ways the agent rationalizes around the skill
2. Plug loopholes in SKILL.md
3. Re-verify
4. Repeat until stable

---

## Mode 2: EDIT

1. Read the existing SKILL.md completely
2. Identify the problem: undertriggering? overtriggering? wrong execution? missing edge case?
3. Apply fix using Phase 5 writing rules
4. Run Phase 6 verify on the changed behavior
5. Check: did the fix break anything else? (regression)

Common edit patterns:

| Problem | Signal | Fix |
|---|---|---|
| Undertriggering | skill doesn't load | add keywords, trigger phrases, file types to description |
| Overtriggering | loads for unrelated queries | add negative triggers, be more specific |
| Skips body | follows description instead | remove process/workflow from description, keep only triggers |
| Inconsistent output | varies across sessions | add explicit templates, reduce freedom, add scripts |
| Too slow | large context | move detail to references/, cut SKILL.md to <500 lines |

---

## Mode 3: EVAL

TDD-style evaluation. Three stages, run in order.

### Stage 1: Discovery (does it trigger correctly?)

Generate 3 test prompts:
- 3 that SHOULD trigger the skill
- 3 that should NOT trigger (similar-sounding but wrong domain)

Run each in clean session. Score: X/6 correct.

Target: 6/6. If <5/6, rewrite description.

### Stage 2: Logic (does it execute correctly?)

Simulate execution step by step:

For each step in the skill:
1. What exactly is the agent doing?
2. Which file/script is it reading or running?
3. Flag any **execution blockers**: points where the agent must guess because instructions are ambiguous

Run 1 use case end-to-end. Document deviations.

### Stage 3: Edge Cases (does it break?)

Attack the skill:
1. What if a script fails?
2. What if input is malformed?
3. What if the user's environment differs from assumptions?
4. What if multiple skills could trigger simultaneously?
5. What implicit assumptions exist?

Document 3-5 vulnerabilities. Fix the critical ones. Re-run Stage 2.

### Scoring

Rate on 5 axes (1-10 each):

| Axis | What it measures |
|---|---|
| Discovery | triggers correctly, doesn't false-trigger |
| Clarity | instructions unambiguous, no guessing needed |
| Efficiency | token budget respected, progressive disclosure used |
| Robustness | handles edge cases, scripts have error handling |
| Completeness | covers the stated use cases fully |

**Score interpretation:**
- 45-50: production ready
- 35-44: solid, minor improvements
- 25-34: needs work on weak axes
- <25: significant rewrite needed

---

## Mode 4: REVIEW

For assessing third-party skills quickly.

Checklist (pass/fail):

```
[ ] SKILL.md exists, exact case
[ ] Valid YAML frontmatter (name + description)
[ ] name: kebab-case, matches folder, ≤64 chars
[ ] description: ≤1024 chars, no angle brackets
[ ] description has triggers ("Use when...")
[ ] description has NO workflow/process steps
[ ] No README.md inside skill folder
[ ] SKILL.md < 500 lines
[ ] References max 1 level deep
[ ] Scripts tested and executable
[ ] No hardcoded paths/tokens/secrets
```

Then run EVAL Stage 1 (discovery) on the description. Report score + checklist.

---

## Mode 5: PACKAGE

1. Run REVIEW checklist
2. Run validation:

```bash
python scripts/quick_validate.py <skill-folder>
```

3. If valid, package:

```bash
python scripts/package_skill.py <skill-folder> [output-dir]
```

Creates `skill-name.skill` (zip with .skill extension).

4. Final check: unzip in temp dir, verify structure intact

---

## Quick Reference

### Skill categories (pick one)

1. **Document/Asset Creation** - consistent output (docs, designs, code)
2. **Workflow Automation** - multi-step processes with methodology
3. **MCP Enhancement** - workflow guidance on top of tool access

### File purposes

| Directory | Loaded into context? | Purpose |
|---|---|---|
| SKILL.md | yes, on trigger | brain - instructions |
| references/ | on demand | detailed docs, schemas, APIs |
| scripts/ | executed, not loaded | deterministic operations |
| assets/ | never loaded | templates, images for output |

### Progressive disclosure budget

| Level | When loaded | Budget |
|---|---|---|
| Frontmatter | always (system prompt) | ~100 words |
| SKILL.md body | on trigger | <500 lines / <5K words |
| Bundled resources | on demand | unlimited (scripts run without context) |

### Description formula

```
[One sentence: what it does] + Use when [specific triggers, symptoms, file types]. + Do NOT use for [negative triggers].
```

### Success metrics

| Metric | Target |
|---|---|
| Triggering accuracy | ≥90% (5/6 in discovery test) |
| Workflow completion without user correction | ≥80% |
| Token consumption vs. baseline | ≤50% |
| Failed API/script calls | 0 |
