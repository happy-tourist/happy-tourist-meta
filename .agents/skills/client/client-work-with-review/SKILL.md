---
name: client-work-with-review
description: >-
  Use when reviewing Vue frontend code, local changes, PRs, or components for
  bugs, regressions, performance issues, and UI overflow risks in the
  happy-tourist client. Complements client-verify-code (skill compliance) and
  client-align-code (requirements gaps) — this skill focuses on defect hunting
  and analogue-backed risk, not OpenSpec completeness or style-guide tidy-ups.
---

# Work With Review

Read-only **bug / regression / risk** review of happy-tourist client changes (or named paths). Deliver findings in chat. Does **not** replace `client-verify-code` (skills/Vue/DRY) or `client-align-code` (постановка vs code).

**Runtime:** `happy-tourist.github.io` via projects-map key `client`.

## Hard Boundary

- Do **not** edit, create, delete, format, or commit files unless the user explicitly asks to apply fixes.
- Do **not** re-run a full verify/align audit — point to those skills if the issue is compliance or requirements gap.
- Prefer evidence: file path + line + short quote; cite a nearby analogue path when recommending a safer pattern.

**Report language: Russian** when the user writes in Russian.

## When To Use

- «посмотри на баги / регрессии», PR review, «что может сломаться», overflow / layout risk
- After implementation, before or beside verify/align — complementary lens

## Scope

Default: branch vs merge-base **plus** staged/unstaged/untracked in the client repo (same collection idea as verify). Narrow to paths the user names.

Focus on:

1. **Bugs / broken paths** — null deref, wrong guard, missed await, race with Colyseus attach, dead click handlers
2. **Regressions** — removed behavior, renamed store/action without callers, broken route meta, HUD/board overlays covering targets
3. **Performance** — unnecessary deep watches, thrashing re-renders, unbounded lists without virtualization where analogues use it
4. **UI overflow / layout** — text overflow, overlapping Quasar overlays, sticky HUD covering board affordances, mobile breakpoints ignored where peer pages handle them

Skip: pure style-guide order, OpenSpec SC coverage, “skill X says Y” restatements (→ verify/align).

## Workflow

1. Resolve client root; collect changed files.
2. Skim diff + open high-risk files (pages, stores, board/HUD, router).
3. Grep callers for renamed public surfaces.
4. Compare with one nearby analogue when suggesting a fix pattern.
5. Report. Stop (or apply only if asked).

## Output

```markdown
## Review
### Bugs
- `path` Lnn — <что ломается>; аналог: `path` (если есть)
- нет

### Regressions
- …
- нет

### Performance / overflow
- …
- нет

### Notes
- optional pointers to `client-verify-code` / `client-align-code` if out of scope here
```

Each item: actionable, with path. No long diff dumps.

## Relation To Other Skills

| Skill | Difference |
|-------|------------|
| `client-verify-code` | Skill compliance + Vue/DRY tidy |
| `client-align-code` | Code vs OpenSpec / analogues / test readiness |
| `verify-mock` | Pixel / Visual Spec vs mock |
| `client-work-with-structure` | Where to place files; passthrough gate |
