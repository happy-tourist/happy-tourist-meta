## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Секции 1–12 (dual flow + staff/delete follow-up) уже в runtime. Этот revision — **UX affordances**: коллекция (клик/корзина/Edit), live Edit при членстве, `inCollection`, pending-author exception. Чеклист — `tasks.md` §13+.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills.

## Goals / Non-Goals

**Goals:**

- Список коллекции: явные actions без ложной навигации; корзина + confirm.
- Live page: Edit когда пак в коллекции; скрыт при чужом pending; автор pending видит Edit.
- Collect button отражает реальное членство (`inCollection`).

**Non-Goals:**

- Менять серверные ACL edit/submit кроме payload для UI; block UI; wipe published.

## Decisions

### D1–D26: Prior (реализовано)

Dual flow, D1′ locks, staff redirect, snapshot fix, author delete unpublished, hide block UI — см. предыдущие revision. **D23 (collection-only Edit, no live Edit) superseded by D27.**

### D27: Live Edit when in collection (replaces D23)

```
live pack page
  |
  +-- NOT in collection --> no Edit (view + add-to-collection only)
  |
  +-- in collection
        |
        +-- any pending answers|tasks whose author != me --> hide Edit
        |
        +-- no pending OR I am author of pending --> show Edit
              (gate login/verify on enter editor, like create)
```

- Collection list **keeps** Edit icon (always → editor route).
- Catalog / stranger live view: no Edit (not in collection).

### D28: Collection list interaction

- Row click → live view if `hasLive`, else editor (draft-only).
- Side icons: Edit → `content-pack-edit`; trash → confirm remove-from-collection.
- MUST stop propagation so icon clicks never fire row navigation.
- Remove icon: Material **`delete`** (корзина), not `remove_circle_outline`.

### D29: `inCollection` on live GET

- `GET /api/content/packs/:id` includes `pack.inCollection: boolean` for the caller.
- Optional (same response or adjacent): enough pending hints for D27 UI, e.g. `pendingAnswersAuthorId` / `pendingTasksAuthorId` (or boolean `canShowEdit` computed server-side). Prefer explicit flags the client can reason about.
- Client: derive collect button from `inCollection` (not ephemeral local `added` reset on load).
- After successful add → set true; after remove (from list) membership updates on next live visit via GET.

### D30: Docs

- Skills/AGENTS: live Edit when in collection + pending-author exception; trash icon; `inCollection` on live; collection click isolation.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Quasar `q-item` `:to` + child buttons | D28: `@click.stop` / separate non-link row body; verify Edit with `hasLive` |
| Pending flags missing on live GET | D29: enrich payload; mocha SC-PACK-61… |
| Users expect Edit while foreign pending | D27: hide; author path unchanged |

## Migration Plan

- Server enrich live GET → client collection + live pages → docs.
- No DB migration.

## Open Questions

- Нет (explore: D1 in-collection Edit, D2 trash, D3 pending author keeps Edit, row→live, collect button).

Чеклист — `tasks.md` §13+.
