## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Секции 1–21 уже в runtime. Этот revision — **cascade карточек ↔ задания без ложного D1′-lock**, confirm copy, жёлтая подсветка task/set, слоты на строке задания, sync статусов под заголовком. Чеклист — `tasks.md` §22+.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills.

## Goals / Non-Goals

**Goals:**

- putDraft после content-change/delete с cascade-слотами **успешен**; `answersDirty`; submit answers доступен.
- Cascade только при реальном сбросе ≥1 слота; description-only и unused delete — без правок заданий.
- Confirm delete: published vs draft по `hasLive`.
- Жёлтый task + task-set, пока после cascade есть дыры; слоты видны на строке задания.
- Subtitle answers/tasks = те же фазы, что метки списка.

**Non-Goals:**

- Ослаблять D1′ для ручного remap слотов / edit questions при dirty answers без open answers.
- Staff hub slot preview.

## Decisions

### D1–D42: Prior (реализовано)

Dual flow, D1′, tasks-only staff, reject-stays / my-moderation / three-phase (D37–D41), docs D42.

### D43: Cascade save is not a tasks edit under D1′

```
putDraft answers change (content / delete card)
  |
  +-- server MAY clear slots that referenced changed/deleted card (SC-PACK-07)
  +-- that side-effect MUST NOT count as tasksChanging for answers_dirty lock
  |
  +-- manual slot/question/set edits while answers dirty + no open answers
        --> still 409 answers_dirty (D1′ unchanged)
```

- Client SHOULD NOT pre-clear slots before save in a way that trips the lock; prefer server clear + return cleared draft, or treat cascade-equivalent body as exempt.
- After successful save: `answersDirty=true`; answers submit enabled when minima met.

### D44: When cascade applies

```
content change of card A  --> clear slots referencing A
delete card A             --> clear slots referencing A (if any)
description-only of A     --> NO slot clear; answers still dirty
delete A with zero refs   --> NO task structure change
```

- «Нужно править задания» только если после операции сбросился ≥1 слот.

### D45: Delete confirm copy

```
hasLive (in catalog) --> «Карточка будет удалена из опубликованного набора…»
!hasLive             --> «…из черновика…» (current wording OK)
```

### D46: Yellow highlight (task + set)

```
after cascade cleared slots
  |
  +-- mark each affected task (has empty slot from cascade) yellow
  +-- mark parent task-set yellow while any such task remains
  |
  until editor fills those slots (or otherwise removes the need to edit)
```

- Do NOT yellow individual slot chips (empty already visible).
- Staff pages: out of scope for this highlight.

### D47: Task-row slot preview

- On the **task list** (task-set page): each task row MUST show its answer slots (text / empty) — strengthen existing caption into clear slot affordance.
- Answers-page nested set list: no requirement to dump all slots (set-level yellow mark is enough).

### D48: Page subtitle = list mark vocabulary

- Cards page subtitle and answers cycle marks: same three-phase strings as list badges.
- Tasks page subtitle and per-set/task marks: same vocabulary (`ожидает отправки` / `на модерации` / `нужна доработка`).
- No divergent «Одобрен» vs «Одобрено» / empty subtitle when list shows a phase.

### D49: Docs

- Skills/AGENTS: cascade save, confirm hasLive, yellow task/set, task-row slots, status sync.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Exempting any slot nulling weakens D1′ | Exempt only cascade from card content/delete; structural key still blocks manual remaps |
| Stale yellow after unrelated empty slots | Tie highlight to slots cleared by last answers cascade / cards that no longer exist |
| Client still pre-clears and trips lock | D43: fix client and/or server; mocha covers both paths |

## Migration Plan

- Server lock/cascade → client confirm/highlight/slots/status → mocha → docs.
- No DB migration.

## Open Questions

- Нет (explore D1–D3 / Q1–Q4 closed).

Чеклист — `tasks.md` §22+.
