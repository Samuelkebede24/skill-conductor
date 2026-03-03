# Architectural Patterns for Skills

## Pattern 1: Sequential Workflow

Best for: onboarding, pipeline, multi-step processes.

```markdown
## Step 1: [Action]
Call tool/script. Expected output: [describe success].

## Step 2: [Action]  
Depends on Step 1 output. If [condition], skip to Step 3.

## Step 3: [Action]
Validate result. On failure: [rollback instructions].
```

Key: explicit ordering, dependencies between steps, validation at each stage, rollback on failure.

## Pattern 2: Iterative Refinement

Best for: content generation, report creation, quality-sensitive output.

```markdown
## Draft
Generate initial output.

## Quality Check
Run `scripts/validate.py`. Criteria:
- [ ] Required sections present
- [ ] Data validated
- [ ] Formatting consistent

## Refine
Address each issue. Re-validate. Repeat until all criteria pass.
Max iterations: 3. If still failing, report issues to user.
```

Key: explicit quality criteria, iteration cap, validation scripts.

## Pattern 3: Context-Aware Selection

Best for: same goal achievable through different tools/approaches.

```markdown
## Decision Tree
- If [condition A]: use approach X (see references/approach-x.md)
- If [condition B]: use approach Y
- Default: use approach Z

## Execute
Follow selected approach. Explain choice to user.
```

Key: clear decision criteria, transparency about choices, fallback options.

## Pattern 4: Domain Intelligence

Best for: compliance, specialized knowledge, expert systems.

```markdown
## Pre-check
Apply domain rules before acting:
- Rule 1: [check]
- Rule 2: [check]
If any fail: flag for review, do NOT proceed.

## Execute
Only if pre-check passed.

## Audit Trail
Log all decisions and checks.
```

Key: domain expertise in logic, compliance before action, documentation.

## Pattern 5: Multi-Service Coordination

Best for: workflows spanning multiple MCPs/APIs.

```markdown
## Phase 1: [Service A]
Fetch/create in Service A. Store result.

## Phase 2: [Service B]  
Use Service A output. Create in Service B.
Validate before proceeding.

## Phase 3: [Notification]
Notify relevant channels with summary.
```

Key: clear phase separation, data passing between services, validation between phases.

---

## Anti-patterns

1. **Wall of text** - paragraphs instead of steps. Agent skims and misses detail
2. **Assumed context** - "use the standard approach" without defining it
3. **Synonym cycling** - template/boilerplate/scaffold for same concept
4. **Hidden prerequisites** - required tools/envs not mentioned
5. **Description as manual** - workflow in description → agent skips body
