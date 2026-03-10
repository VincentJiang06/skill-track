# skill-rules-designer

A Claude Code skill that analyzes existing skills and designs optimal `rules/` file structures.

This is a focused extension to Anthropic's official [Skill Creator](https://github.com/anthropics/anthropic-agent-skills) — it picks up where Skill Creator leaves off by helping you architect the internal structure of a skill once the initial draft exists.

---

## What problem does it solve?

Skill Creator helps you write a skill and run evals. But as skills grow, a single `SKILL.md` becomes a problem:

- Every invocation loads the **entire** file — including sections that only apply in specific situations
- Long files are harder to maintain and iterate
- Steps that require templates or checklists get re-invented from scratch on each run
- Vague instructions produce inconsistent behavior across Claude instances

`skill-rules-designer` analyzes your skill and gives you a concrete plan to fix this using `rules/` files.

---

## How `rules/` files work

Claude Code skills support a `rules/` directory alongside `SKILL.md`. Files placed there can be:

- **Always loaded** — part of the skill's permanent context (no token savings, but cleaner structure)
- **Conditionally loaded** — loaded only when `SKILL.md` tells Claude to read them, based on what the user actually needs

This conditional loading is the key mechanism. A skill that handles 5 different output modes doesn't need all 5 mode specs in context on every invocation — only the one the user selected.

---

## Four operations

`skill-rules-designer` analyzes a skill across four dimensions:

| Operation | What it does | Token impact |
|-----------|-------------|-------------|
| **Compress** | Moves verbose content from `SKILL.md` to a rules file | None — cleaner structure only |
| **Encapsulate** | Moves optional features to rules files loaded conditionally | Reduces per-invocation cost |
| **Enrich** | Creates new template/checklist files for steps done ad-hoc | Improves consistency |
| **Harden** | Rewrites vague instructions to be precise and unambiguous | Improves reliability |

All operations are **lossless** — content is moved, never deleted. The full skill logic is always recoverable from the file set.

---

## Usage example

Say you have a `course-study` skill with a 268-line `SKILL.md` and no `rules/` files. You run:

> "帮我把这个 skill 的 rules 结构设计一下。路径：`/Users/you/.claude/skills/course-study`"

The skill reads your `SKILL.md` and produces a restructuring plan:

```
## Restructuring Plan — course-study

Current: SKILL.md (268 lines) + 0 rules files
After:   SKILL.md (~60 lines) + 7 rules files

### Compress
→ Phase 1 extraction procedure (~50 lines) → rules/phase-extract.md
→ Phase 2 synthesis procedure (~55 lines)  → rules/phase-synthesize.md
→ Phase 3 expansion procedure (~55 lines)  → rules/phase-expand.md
→ Phase 4 study notes generation (~80 lines) → rules/phase-study.md
→ Curriculum gap analysis (~50 lines)      → rules/subject-coverage.md

### Encapsulate
→ Exam Ready appendix specs (~55 lines)    → rules/exam-ready.md
  Condition: only loaded when user selects Exam Ready mode
  Token savings: ~55 lines on all Standard-mode invocations

→ PDF/CJK font configuration (~100 lines)  → rules/templates.md
  Condition: only loaded when user requests PDF export
  Token savings: ~100 lines on every invocation that doesn't need a PDF

### Enrich
→ New file: rules/intake-guard.md
  Contains: Phase 0 state machine with safe defaults and a hard close rule
  Replaces: prose aspiration "keep intake to one exchange" (currently unenforceable)

→ New file: rules/progress-tracker.md
  Contains: TodoList protocol with state transitions per lecture
  Replaces: one-sentence "Use TodoList" rule (interpreted inconsistently)

### Harden
1. SKILL.md:60 "Detect WebSearch availability silently."
   Problem: "silently" is undefined — some instances announce the attempt, some don't
   Proposed: "Attempt a single no-op WebSearch call. If it succeeds, set mode = A.
   Report in one line: '✓ Web access available' or '✗ No web access'."

2. SKILL.md:138 "give them deeper treatment"
   Problem: "deeper" is unquantified
   Proposed: "Apply the 'Core / exam-critical' concept weight (all seven sections) to
   every priority topic, even if it would normally be 'Important supporting'."

---
Lossless check: all content currently in SKILL.md will exist in the new file set.

Say "go" to proceed, or give me your changes first.
```

After you say "go", the skill writes all files in the correct order — rules files first, then updates `SKILL.md` to remove extracted content and add module references.

---

## Relationship to Skill Creator

| | Skill Creator | skill-rules-designer |
|--|--------------|---------------------|
| Write initial skill draft | ✅ | ❌ |
| Run evals and benchmark | ✅ | ✅ (v1.1) |
| A/B comparison with scorecard viewer | ✅ | ✅ (v1.1) |
| Optimize trigger description | ✅ | ❌ |
| Design `rules/` file structure | ❌ | ✅ |
| Identify encapsulation opportunities | ❌ | ✅ |
| Add template / resource files | ❌ | ✅ |
| Harden vague instructions | ❌ | ✅ |
| Lossless restructuring guarantee | ❌ | ✅ |

Use Skill Creator to build and benchmark your skill. Use `skill-rules-designer` when the skill is working but needs structural optimization — or to verify that a restructuring didn't degrade quality.

---

## Eval & A/B Comparison (v1.1)

v1.1 adds a full eval workflow so you can verify a restructuring actually improved the skill without degrading output quality.

### What's included

```
evals/evals.json       — 3 pre-built test cases (compress+encapsulate, harden, enrich)
agents/grader.md       — assertion grader agent
agents/comparator.md   — blind A/B comparator agent
agents/analyzer.md     — post-hoc analyzer agent
references/schemas.md  — JSON schemas for all eval artifacts
```

> **Note:** The `agents/` and `references/schemas.md` components are directly based on Anthropic's official [Skill Creator](https://github.com/anthropics/anthropic-agent-skills/tree/main/skills/skill-creator). The grader, comparator, and analyzer agent prompts and the JSON schemas were taken from that project and adapted for this skill's use case.

### Terminal output

Results print directly in the terminal — no browser or server needed:

```
## A/B Comparison — skill-rules-designer
─────────────────────────────────────────────
  Quality (Pass Rate)  Version A: 86%    Version B: 71%    Δ +15%  ✓ A wins
  Token Usage          Version A: 42,500  Version B: 31,000  Δ +37%  ✗ A costs more
  Duration             Version A: 95s    Version B: 78s    Δ +22%  ✗ A is slower
─────────────────────────────────────────────
Per-eval breakdown + assertion-level pass/fail
Analysis notes from post-hoc analyzer
```

### Running an A/B comparison

Described in full in `SKILL.md` under "Evaluating the skill". Short version:

1. Spawn executor subagents for `version_a` and `version_b` on all 3 evals in the same turn
2. Capture `total_tokens` + `duration_ms` from each subagent notification → `timing.json`
3. Grade each run with the grader agent → `grading.json`
4. Print the formatted comparison report in the terminal

---

## Installation

Copy the skill folder into your Claude Code skills directory:

```bash
cp -r skill-rules-designer ~/.claude/skills/
```

Claude Code will pick it up automatically on next launch.

---

## Trigger phrases

- "my skill is too long / help me structure my rules files"
- "split this skill / reduce token usage"
- "add a template to my skill"
- "make this rule more precise"
- Show a `SKILL.md` and ask how to improve its structure or efficiency
