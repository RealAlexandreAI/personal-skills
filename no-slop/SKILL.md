---
name: no-slop
description: Detect and fix AI-generated slop in existing code — env/config sprawl, fallback hydra, legacy cruft, swallowed errors, duplicated logic, narrative comments, and decision history leaking into customer-facing docs. Proposes a fix list and confirms with the user before applying any change. Use when asked to clean up, de-slop, fix slop, or remove AI slop from code, a diff, or a PR.
---

# No Slop

Slop is not "AI-written code." It is code that compiles, passes tests, and still can't be trusted: every failure grows a new fallback, every migration keeps the old path "just in case," every doc explains why instead of how. Fixing it means deleting the extra paths, not adding a new one on top.

## Scope

| Scope | Meaning | Resolve with |
| --- | --- | --- |
| Uncommitted | working tree: staged + unstaged, not yet committed | `git diff HEAD` |
| Unpushed | local commits ahead of the remote branch | `git diff @{u}...HEAD` |
| PR / branch | this branch vs. its base (merge-base diff) | `git diff $(git merge-base <base> HEAD)...HEAD`, or `gh pr diff` if a PR is open |
| Whole repo | everything, no diff | scan the tree directly |

If the user names a scope, use it. Otherwise resolve in this priority order and use the first one that's non-empty: uncommitted → unpushed → PR/branch → whole repo. State the resolved scope at the top of your output so the user can override it.

## Workflow

1. **Scope** — resolve using the table above.
2. **Detect** — scan for the signals below, each with file:line.
3. **Propose** — present a fix list ordered delete → consolidate → rewrite, using the format below. **Stop and get explicit confirmation before editing anything.**
4. **Apply** — only the confirmed items, in that order.
5. **Verify & report** — run existing tests/build if available; state exactly what you deleted and changed.

## Signals to detect

- **Env & config sprawl** — env var/flag/config knob whose value has one correct answer, or that nothing reads yet.
- **Fallback hydra** — 2+ paths for one operation (`try → catch → retry → default`). Keep exactly one; delete the rest.
- **Legacy & dead branches** — deprecated/legacy/compat code, commented-out code, unreachable branches kept "for safety."
- **Swallowed errors** — catch blocks that return a default or succeed silently, with no logging and no real decision.
- **Duplication** — a helper/type that reimplements something already in the codebase.
- **Narrative comments** — comments that restate the code or narrate a change instead of explaining *why*.
- **Docs audience mismatch** — customer-facing docs (README usage, help text, 对客文档) describing internal technical decisions, rejected alternatives, or decision history.
- **Over-engineering** — abstractions/options built for a hypothetical need, not the asked use case.

## Fix list format (show this before touching code)

```markdown
## Fix list for [scope]
1. [delete] file:line — dead/legacy/fallback branch, why it's safe to remove
2. [consolidate] file:line + file:line — duplicate helpers to merge, keep which one
3. [rewrite] file:line — swallowed error / narrative comment / config to fix

Proceed with all, or pick numbers?
```

Wait for the user's answer before editing anything.

## Fix rules

- Delete before you add — most slop is inert; removing it changes no runtime behavior.
- Collapse fallback chains to the one path that should run; make failure explicit (typed error or clear log), never a silent default.
- When merging duplicates, keep the more-used or better-tested version and update its callers.
- If a customer-facing doc has decision history worth keeping, move it into an internal note (ADR/changelog) instead of just deleting it.
- Never introduce a new env var, fallback, or shim while fixing — that's slop again.

To score/audit code without changing it, use the `slop-taste` skill.
