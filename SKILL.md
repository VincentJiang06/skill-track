---
name: skill-track
version: 1.0.0
description: >
  Git-native skill iteration tool for testing, evaluating, and tracking versions of Claude Code skills.
  Use when you want to test a skill against multiple test cases, debug a single rules module, compare
  skill versions (A vs A₁ vs B), track iteration history with Git, or decide whether a skill needs
  a small bug fix or a full redesign. Complements the official Skill-Creator by adding version
  management, single-module debugging, and structured iteration decisions. Not a skill generator —
  this tool runs your tests, judges the results, manages Git branches and tags, and tells you
  exactly what to fix next.
---

# skill-forge

A Git-native iteration tracker for Claude Code skills. Test → Evaluate → Decide → Track.

## How to trigger

Invoke this skill when the user says things like:
- "test my skill"
- "run evals on this skill"
- "debug this rules file"
- "compare skill versions"
- "start a new iteration round"
- "forge / iterate / improve my skill"

## Entry point

When triggered, follow `rules/intake.md` first.

The full workflow:

```
intake.md          → read target skill, Git init, choose mode
    ↓
test-runner.md     → run test cases sequentially
    ↓
evaluator.md       → local rules check + single batch LLM evaluation
    ↓
iteration.md       → present score, recommend Debug A₁ / Redesign B / Stop
    ↓
git-ops.md         → after user edits: branch, commit, tag, merge
```

Single-module debug shortcut:
```
intake.md → module-debug.md → evaluator.md (module-scoped) → git-ops.md
```

## Rules files

- `rules/intake.md` — entry, read target skill, Git init, mode selection
- `rules/test-runner.md` — sequential test execution per case
- `rules/evaluator.md` — two-layer evaluation (local rules + batch LLM)
- `rules/iteration.md` — decision framework, version naming, user prompt
- `rules/module-debug.md` — single rules file targeted debug mode
- `rules/git-ops.md` — all Git operations, branch/commit/tag/merge standards
