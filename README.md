# skill-track

**A Git-native iteration tracker for Claude Code skills.**

Test your skill → evaluate results → decide what to fix → track every version with Git.

Not a skill generator. Not a one-click optimizer. A structured workbench for the real development loop: run → judge → fix → repeat — with every iteration committed and tagged.

---

## What problem it solves

When you're developing a Claude Code skill, the usual workflow is informal:
- Run it a few times, feel like it's working
- Change something, run it again
- Lose track of which version was better
- Have no idea which rules file is causing the issue

skill-track adds structure to that loop:
- **Sequential test execution** across defined test cases
- **Two-layer evaluation**: local rule checks (free) + single batch LLM call (cheap)
- **Iteration decision**: tells you whether you need a small bug fix (A₁) or a full redesign (B)
- **Git version tracking**: every round gets a branch, commit, and tag — no version is ever lost

---

## Who is it for

- **Skill developers** who are iterating on a skill and want to track progress
- **Teams** who want a repeatable test process instead of ad-hoc "let me try it"
- **Anyone** who has changed a rules file and wants to quickly verify it didn't break anything

---

## Two modes

### Full test mode
Runs all test cases against the complete skill. Use for:
- End-of-session "is this ready?" checks
- Comparing two major versions (A vs B)
- Before tagging a release

### Module debug mode
Runs 3–5 targeted test cases against a **single rules file** in isolation. Use for:
- Verifying a small edit to one module
- Debugging why a specific phase isn't working
- 80% faster than full test when you only changed one file

---

## How it works

```
1. Point skill-track at your skill's directory
2. Choose full test or module debug
3. Select or describe test cases
4. skill-track runs each case sequentially
5. Results are evaluated in two layers (0 LLM overhead for structural checks)
6. You get a score + a concrete recommendation:
     [A₁] Debug small fix — patch the bugs, keep core logic
     [B]  Redesign — rethink the approach
     [✓]  Stop — it's ready
7. You edit your files. skill-track commits, branches, and tags the new version.
8. Repeat.
```

---

## Version naming

| What happened | You see | Git tag | Branch |
|--------------|---------|---------|--------|
| Initial version | A | `vA` | `main` |
| Bug fix | A₁ | `vA.1` | `fix/vA.1` |
| Another bug fix | A₂ | `vA.2` | `fix/vA.2` |
| New approach | B | `vB` | `feat/vB` |
| Bug fix on B | B₁ | `vB.1` | `fix/vB.1` |

Every passing version merges to `main` with `--no-ff`. Every failing version's branch is deleted cleanly.

---

## Evaluation cost

| Scenario | Token estimate |
|----------|---------------|
| Full test, 5 cases | ~13k tokens |
| Module debug, 3 cases | ~5.5k tokens |
| Two-round iteration (A → A₁) | ~26k tokens |

One LLM call per evaluation round. Never one call per test case.

---

## Relationship to official Skill-Creator

Anthropic's Skill-Creator handles: test case design, benchmark scoring, auto-improvement.

skill-track handles what Skill-Creator doesn't:

| | Skill-Creator | skill-track |
|--|--------------|-------------|
| Git version history | ❌ | ✅ |
| Single module debug | ❌ | ✅ |
| Debug vs Redesign decision | ❌ | ✅ |
| Full benchmark reporting | ✅ | minimal |
| Auto-generate improved version | ✅ | ❌ (by design) |

They work well together: use Skill-Creator for deep benchmarks, use skill-track for day-to-day iteration tracking.

---

## File structure

```
skill-track/
├── SKILL.md
├── README.md
└── rules/
    ├── intake.md          # entry point, Git init, mode selection
    ├── test-runner.md     # sequential test execution
    ├── evaluator.md       # two-layer evaluation
    ├── iteration.md       # decision logic, version naming
    ├── module-debug.md    # single module targeted debug
    └── git-ops.md         # Git branch/commit/tag/merge standards
```

---

## Changelog

### v1.0.0
- Initial release
- Full test mode + module debug mode
- Two-layer evaluation (local rules + single batch LLM)
- Git-native version tracking with branch/commit/tag/merge workflow
- Iteration decision: Debug (A₁) / Redesign (B) / Stop
