## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Секции 1–9 (dual flow + UX polish) уже в runtime. Этот revision — **follow-up**: staff navigation, marks/snapshot bug, collection gates, D1′ lock, hide block UI, author delete unpublished. Чеклист — `tasks.md` §10+.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills.

## Goals / Non-Goals

**Goals:**

- Staff не видит мёртвую tasks-страницу после approve; список без «уже промодерировано».
- Метки needsModeration согласованы со статусом «одобрено».
- Edit только из коллекции; remove из коллекции с confirm.
- Pending-author A может править задания при dirty answers; чужие — нет.
- Автор удаляет неопубликованный пак или task set (в т.ч. approved liveTasks).
- Block buttons убраны с UI.

**Non-Goals:**

- Staff delete / block anytime / publish-pack wipe; partial submit; новый transport.

## Decisions

### D1–D20: Prior (реализовано)

Dual flow, statuses/threads, tasksDirty/marks, staff nested, quiet autosave — см. предыдущие revision. Ниже — **добавления/правки**.

### D5′′ / D1′: Dirty answers + кто правит tasks

- Пока answers **dirty** и **нет** answers-pending: create/edit tasks запрещены всем (сначала submit answers).
- Пока answers **pending** под автором A (в т.ч. если A снова сделал answers dirty): **A MAY** create/edit tasks; **другие** MUST NOT.
- Submit tasks по-прежнему: чужие при answers pending запрещены; A MAY submit tasks.
- UI/i18n отражают исключение для A.

### D21: Sticky needsModeration after approve (bugfix)

Корневая причина: `detachAuthorDraftIfPinned` → `copyRevision` **перегенерирует id**, а `last_*_snapshot` остаётся со старыми id → все marks dirty.

Варианты фикса (достаточно одного):

- При detach после approve: переписать `last_tasks_snapshot` / `last_answers_snapshot` из **нового** draft payload; и/или
- `copyRevision` сохраняет стабильные entity id где возможно.

После фикса: статус «одобрено» и marks «нужна модерация» не противоречат без реальных правок.

### D22: Staff after approve tasks

```
approve tasks --> redirect --> answers hub
hub task-set list: omit sets that are fully moderated
                   (no pending tasks work left for that set / pack tasks approved)
nested tasks page: do not reload via getLivePack when pack not public
                   (use liveTasks / pending preview; no pack_not_public dead-end)
```

- Убрать redundant «Открыть задания» (навигация = клик по строке списка, пока строка есть).

### D23: Collection-only edit affordance

- Кнопка Edit на public/live pack page — **убрать**.
- Edit остаётся в «Моей коллекции» (+ deep-link в editor для уже имеющих коллекцию).
- Server `not_in_collection` без изменений.

### D24: Remove from collection + confirm

- Иконка/действие «убрать из коллекции» остаётся; перед вызовом API — confirm dialog.
- Это **не** удаление пака.

### D25: Hide block UI

- Убрать кнопки block/unblock со staff pages в этом раунде.
- Server endpoints MAY остаться; новых surface для block anytime не делать.
- Staff/public delete опубликованного — later.

### D26: Author delete unpublished (D5′=A, D6, D7)

**Неопубликованный** = нет live answers / не в каталоге (`liveRevisionId` null).

Только `createdBy`:

1. **Удалить пак** («набор карточек»): hard wipe pack + drafts + revisions + collections rows + moderation requests/messages (**cascade** pending).
2. **Удалить task set**: убрать set из draft **и** из `liveTasksRevisionId` content, даже если tasks уже staff-approved. Ключ публикации — answers, не tasks.

Опубликованный пак: author delete out of scope (этот раунд).

### D11′′: Docs

- Skills/AGENTS: collection-only edit, confirm remove, author delete unpublished, D1′, staff redirect, no block buttons.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| copyRevision id remap | D21 snapshot rewrite / stable ids + tests |
| Author deletes set after tasks approve | D7: sync liveTasks; answers still gate catalog |
| Hide block while SC-PACK-25 exists | D25: server capability retained; UI deferred; document in spec |

## Migration Plan

- Server fix approve/detach + delete endpoints → client UX → hide block → docs.
- Dev DB: existing false-dirty drafts очищаются после следующего approve или ручного resubmit.

## Open Questions

- Нет (explore follow-up закрыт: D1′, D2′, D5′=A, D6, D7, cascade).

Чеклист — `tasks.md` §10+.
