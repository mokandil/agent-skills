# agent-skills

Installable agent skills for safer AI-assisted software development.

## Skills

| Skill | Problem it solves | Install |
|-------|------------------|---------|
| [agent-scope-guard](./agent-scope-guard/SKILL.md) | Scope creep, silent regressions, feature erasure during AI coding sessions | `npx skills add mokandil/agent-skills@agent-scope-guard` |

## Install all

```bash
npx skills add mokandil/agent-skills
```

## Install one skill

```bash
npx skills add mokandil/agent-skills@agent-scope-guard
```

---

## agent-scope-guard — benchmark

A/B tested against Claude Sonnet 4.6 across three real-world scenarios: a backend refactor with "while you're in there" pressure, a component redesign (feature erasure risk), and a pre-commit diff review with out-of-scope files in the diff.

| Configuration | Assertions passed | Pass rate |
|---|---|---|
| with `agent-scope-guard` | 16 / 16 | **100%** |
| without skill (baseline) | 10 / 16 | 62% |
| Delta | +6 | **+38pp** |

### What the skill changes

Without the skill, agents make the right call on individual requests (refuse a bad import cleanup, block a bad commit) but skip the structural checkpoints:

- No explicit scope declaration before editing — adjacent files get touched silently
- No feature inventory before redesigns — PRESERVE classification never happens, erasure risk is high
- No `git diff <base-branch>` before committing — surprises only surface after the fact

With the skill, all three checkpoints are enforced on every task.

### Biggest gain: backend refactor (+60pp)

The "while you're in there" scenario — user asks to add a feature plus clean up imports plus address a TODO in the same request. Without the skill, agents refused the bad requests but never declared scope and never mentioned diff review. With the skill, agents produced a full PRESERVE table, declared scope explicitly, and included a post-edit diff checklist.

### Scenario: component redesign (+33pp)

"Redesign this component — make it cleaner." Without the skill, agents inventoried features but never classified them as PRESERVE vs INTENTIONALLY CHANGING. With the skill, agents produced an exhaustive PRESERVE list (60+ methods/behaviors), caught a file path discrepancy before touching anything, and planned a diff verification step keyed to specific named items.

### Scenario: pre-commit diff review (+20pp)

Smallest gap — projects with a strong `CLAUDE.md` already enforce diff review culturally. The skill adds the explicit "I will not silently revert" guarantee and per-file confirmation questions that baseline agents sometimes skip.
