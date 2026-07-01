---
name: slop-taste
description: Systematically score and audit code, diffs, PRs, or docs for AI-generated slop — env/config sprawl, fallback hydra, legacy cruft, and decision history leaking into customer-facing docs. Use when the user asks to review, audit, grade, or check code or documentation for AI slop, vibe-coded smell, or quality after AI generation.
---

# Slop Taste

AI slop is a code/product smell, not a claim about authorship: code that compiles, passes tests, and still can't be trusted or safely extended. Audit with evidence, not vibes.

## Scope

| Scope | Meaning | Resolve with |
| --- | --- | --- |
| Uncommitted | working tree: staged + unstaged, not yet committed | `git diff HEAD` |
| Unpushed | local commits ahead of the remote branch | `git diff @{u}...HEAD` |
| PR / branch | this branch vs. its base (merge-base diff) | `git diff $(git merge-base <base> HEAD)...HEAD`, or `gh pr diff` if a PR is open |
| Whole repo | everything, no diff | scan the tree directly |

If the user names a scope, use it. Otherwise resolve in this priority order and use the first one that's non-empty: uncommitted → unpushed → PR/branch → whole repo. State the resolved scope at the top of your report so the user can override it.

## Workflow

1. **Scope** — resolve using the table above.
2. Score each category below 0–5, citing file:line evidence for anything above 1.
3. Report using the template.

## Scoring rubric

- **0 Sharp** — one path per operation, no dead weight, docs match audience.
- **1 Pragmatic** — a couple of justified adapters/flags with clear contracts.
- **2 Slight slop** — one unnecessary env var, one duplicated helper, one narrative comment.
- **3 Visible slop** — multiple categories show real smell; a reviewer has to guess which path runs.
- **4 Heavy slop** — several categories at 3+; failure behavior is undocumented and inconsistent.
- **5 Slopmax** — nobody can say which of N paths runs in production.

Overall score = average of categories below. Any single category at 5 is a blocker regardless of the average.

## Categories

| Category | What to check | Strong slop evidence |
| --- | --- | --- |
| Env & config sprawl | new env vars, flags, config knobs | added "just in case," nothing reads them yet, or the value has only one correct answer |
| Fallback hydra | try/catch chains, retries, resolvers | 2+ fallback paths for one operation; final catch returns a plausible-looking default |
| Legacy & dead branches | deprecated/legacy/compat code | old path kept "for safety" after a migration; commented-out or unreachable code |
| Error honesty | catch blocks, broad excepts | swallowed exceptions, no logging, silent success on failure |
| Duplication | new helpers/types vs. existing ones | a near-identical function already exists two directories away |
| Comment/copy noise | comments, docstrings | narrates the diff ("now we check X"), restates the code, gives no *why* |
| Docs audience fit | README, help text, 对客文档 vs. internal notes | customer-facing doc explains internal technical decisions, rejected alternatives, or decision history instead of usage |
| Structural erosion | file/function size, ownership | one file or function knows every subsystem; god class/function |

## Report template

```markdown
## Verdict
Slop score: X/5 — one direct sentence.

## Scorecard
| Category | Score | Evidence |
| --- | ---: | --- |

## Top risks
1. file:line — the finding and why it matters (ranked by severity, not by category order)

## Clean target
One or two sentences describing the boring, single-path version.
```

This is a grade with evidence, not an edit. Don't change any code while running this skill.

## Rules

- Evidence only: cite file paths and symbols, not impressions.
- Rank by severity: a swallowed error in a payment path outranks a narrative comment.
- Size ≠ slop. A large file can be sharp if it has one clear owner and no dead paths.
- To fix the findings, run the `no-slop` skill on the same scope — it will propose and confirm a fix list before touching code.
