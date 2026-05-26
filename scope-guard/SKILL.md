---
name: scope-guard
description: Use when starting any coding task or before committing changes — enforces scope declaration before first edit, feature inventory before redesigns, and diff review before every commit to catch scope creep, feature erasure, and silent regressions.
---

# Scope Guard

Three failure modes ruin AI-assisted coding sessions — especially on large, long-running projects:

1. **Scope creep** — agent touches adjacent files that weren't asked for ("while I'm in there...")
2. **Feature erasure** — during a redesign, existing features/config/behaviors silently disappear; user discovers this sprints later
3. **Silent regression** — a change to a shared module silently breaks behavior that was working, with no test catching it

This skill enforces three hard checkpoints that prevent all three.

---

## Checkpoint 0: Feature Inventory (before redesigning existing code)

**Applies when:** the task is a redesign, refactor, or significant modification of an existing component, page, or module — not a greenfield addition.

Before writing a single line, read the file and list everything that currently exists:

> "This file currently has: [feature A], [config option B], [behavior C], [UI state D]. I will preserve ALL of these unless you explicitly tell me to remove one."

Then, for each item in the list, mark it:
- **PRESERVE** — must survive unchanged after my edits
- **INTENTIONALLY CHANGING** — part of the task (confirm with user if unsure)

After making changes, verify every PRESERVE item is still present and working before committing. If any PRESERVE item disappeared from the diff, stop — do not commit.

**Why this matters:** Redesign tasks are the highest-risk scenario. The agent rewrites layout/structure and silently drops a setting that was deliberately there. The user doesn't notice for several sprints and has to excavate git history to recover it.

---

## Checkpoint 1: Scope Declaration (before first edit)

Before touching ANY file, state out loud:

> "I will ONLY modify: `[file A]`, `[file B]`. I will NOT change: `[adjacent area X]`, `[test files]`, `[config]`."

Then stop. If the user pushes back, update the list before proceeding.

**What counts as "in scope":** Only files the task description explicitly names or whose change is logically required to make the stated task work — nothing else.

**What is NEVER in scope without explicit unlock:**
- Files with a `# TODO` comment near the area you're working in
- Test files (unless the task says "add tests")
- Config files (unless the task says "update config")
- Shared utility or type files you're only importing from
- Any file you're touching because it "should probably be consistent"

---

## Checkpoint 2: Diff Review (before every commit)

Before staging or committing anything, run:

```bash
git diff <base-branch> --stat        # ALL changes that will land when you merge — full picture
git diff <base-branch>               # full line-by-line diff
```

**Use `git diff <base-branch>`, not `git diff HEAD`.** They are different:
- `git diff HEAD` — only uncommitted local changes (misses already-staged surprises)
- `git diff <base-branch>` — everything that will land on the base branch when you merge

The base branch is usually `develop` or `main`. When unsure: `git merge-base HEAD $(git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@')`.

Scan every file in the diff. Ask two questions:
1. **Is this file in my declared scope?** If no → stop and surface it.
2. **Are lines being deleted that were in my PRESERVE list?** If yes → stop and surface it.

Report unexpected findings to the user:
> "I notice `[file/feature]` changed in a way that was not in scope. Specifically: `[what changed]`. Should I revert this, keep it, or split it into a separate commit?"

Do NOT silently revert. Do NOT silently keep it. Always surface it.

---

## Red Flags — Stop and Declare Scope

| Thought | What it actually means |
|---------|----------------------|
| "While I'm in there, I'll also fix..." | Out of scope. Stop. |
| "This adjacent file needed a small tweak" | Declare it or don't touch it. |
| "The TODO comment right here deserves attention" | TODOs are not task assignments. |
| "I should keep this consistent with the rest" | Consistency is a separate task. |
| "The user said 'fix it properly'" | Fix the stated thing properly. Nothing more. "Properly" modifies HOW, not WHAT. |
| "The user said 'while you're in there'" | That phrase is not an authorisation to expand scope. |
| "It's just a config constant, harmless" | Harmless changes still need to be declared. |
| "I verified by reading the file, no need to diff" | Reading ≠ diffing. Run the diff. |
| "The change is too small to matter" | Small undeclared changes cause silent regressions. |
| "I'm redesigning this component anyway" | Redesign = highest risk of feature erasure. Run Checkpoint 0. |
| "That setting was probably a default" | It was deliberately set. Preserve it until told otherwise. |

---

## Silent Regression Guard

After any change to a **shared module** (utils, types, state, constants), before committing:

1. Run the test suite: confirm nothing broke
2. Run `git diff <base-branch>` and check: does any non-scope file appear?
3. If yes → surface it before committing

A file you didn't intend to touch showing up in the diff is the signature of a silent regression.

---

## The Iron Law

**No commit without a diff review. No redesign without a feature inventory. No edit without a scope declaration.**

- Skipping Checkpoint 0 on a redesign = features will be silently erased; user discovers this sprints later
- Skipping Checkpoint 1 because "it's obvious what I'll touch" = silent regression waiting to happen
- Skipping Checkpoint 2 because "the change is small" = the exact scenario where surprises live
- "I read the file before and after" does not replace `git diff`
- These checkpoints take 2 minutes. A missed feature takes hours to recover from git history.
