## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Секции 1–15 уже в runtime. Этот revision — **staff tasks-only**: очередь + тот же hub (ответы → task sets), approve только tasks. Чеклист — `tasks.md` §16+.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills.

## Goals / Non-Goals

**Goals:**

- Tasks-only pending виден в staff queue рядом с answers-pending.
- Hub UX как сейчас: ответы (весь набор) сверху → список task sets → nested tasks.
- Tasks-only: нет кнопок approve answers; только tasks moderation.
- Answers+tasks pending: прежний порядок (tasks live before answers approve).

**Non-Goals:**

- Менять D1′ dirty edit/submit locks, seed snapshots, collection/live Edit polish.
- Отдельная вкладка очереди (один список).

## Decisions

### D1–D30: Prior (реализовано)

Dual flow, D1′ locks, staff answers-only queue (SC-PACK-30/40), snapshot-on-detach, collection/live Edit (D27–D29). **SC-PACK-30/40 «tasks-only invisible» superseded by D34.**

### D34: Staff queue includes tasks-only

```
listPendingPacks
  |
  +-- pending type=answers  --> list item (as today)
  +-- pending type=tasks AND no answers pending --> list item (NEW)
  +-- both answers+tasks pending --> one list item via answers request (nested tasks as today)
```

- Deduplicate by pack: if answers pending exists, list via answers requestId (hasTasksPending flag). Do not double-list the same pack for tasks.
- Tasks-only row: `requestId` = tasks pending id (or stable hub entry id); client opens same hub route with mode `tasksOnly`.

### D35: Same hub layout; tasks-only hides answers actions

```
staff hub (answers pending OR tasks-only)
  |
  +-- answers/cards preview (full pack context)
  |     answers pending: from answers request revision
  |     tasks-only: from live answers (read-only context)
  |
  +-- task-set list (marks / nested link)
  |
  +-- answers pending: approve/reject/cancel answers + thread
  +-- tasks-only: NO answers approve/reject; optional read-only note
  |
  +-- nested tasks page: approve/reject/cancel tasks (unchanged)
```

- After tasks-only approve → leave hub / back to queue (no answers approve step).
- Catalog already live when answers were approved earlier; tasks approve updates liveTasks / merge as today.

### D36: Docs

- Skills/AGENTS: staff queue tasks-only; hub same layout; no answers approve when tasks-only.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Double-list pack with both pendings | D34: prefer answers row |
| Hub route assumes answers requestId | D34/D35: support tasks requestId + `tasksOnly` / detect by request.type |
| Old tests SC-PACK-30/40 | Rewrite to expect tasks-only listed |

## Migration Plan

- Server listPending + preview entry → client staff pages → mocha → docs.
- No DB migration.

## Open Questions

- Нет (T1 same hub, T2 whole pack, T3 tasks-only approve; dirty/marks out of scope).

Чеклист — `tasks.md` §16+.
