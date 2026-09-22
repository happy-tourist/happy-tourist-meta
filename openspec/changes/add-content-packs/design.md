## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Сейчас в runtime: auth + `emailVerified`, роли `user`/`moderator`/`admin`, HTTP support (тикеты, тред, mail через smtp.bz), SQLite/Drizzle custom tables через ensure-at-boot. Peek на поле — stub (`game/board`). Контентных таблиц/API нет.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + при необходимости meta (AGENTS/skills). Чеклист — `tasks.md`.

## Goals / Non-Goals

**Goals:**

- HTTP API + SQLite модель packs / answer cards / task sets / tasks / slots / collections / moderation requests+messages / block flag.
- Client: каталог, коллекция, view/editor, staff queue + thread; modal verify/auth.
- Модерация: pending lock, race на submit, mail автору заявки, cancel staff.
- Difficulty 1–3 persist (задел); tourist room не трогать.

**Non-Goals:**

- Room↔pack, join gate, peek runtime из pack.
- Картинки/медиа; diff-only staff UI; hard delete.
- Colyseus realtime для контента.

## Decisions

### D1: Transport = HTTP + JWT (как support)

- `createEndpoint` / `auth.middleware()`; Pinia store на client через `client.http`.
- Нет Colyseus room для packs.
- Alternative (room sync) — отвергнут: контент не realtime match state.

### D2: Data model (SQLite, ensure-at-boot)

Таблицы (имена уточняются при apply, смысл фиксирован):

| Сущность | Смысл |
|----------|--------|
| `content_packs` | id, title, description, created_by, live revision pointer / status flags, `blocked`, timestamps |
| `content_pack_revisions` или draft blob | live approved snapshot vs pending draft автора заявки |
| `content_answer_cards` | pack-scoped; content + description; stable ids для слотов |
| `content_task_sets` | pack-scoped; author_user_id; optional coauthor labels |
| `content_tasks` | task_set_id; question; difficulty 1\|2\|3; slot order |
| `content_task_slots` | task_id; position; answer_card_id nullable |
| `content_pack_collections` | user_id + pack_id |
| `content_moderation_requests` | pack_id, change_author_id, status pending\|approved\|rejected\|cancelled, thread id |
| `content_moderation_messages` | request_id, author_user_id, author_kind user\|staff, body, created_at |

- Паттерн: как `ensureSupportTables()` — не SchemaSet auto-sync для кастомных таблиц.
- **Live vs pending:** публичные GET отдают только approved live snapshot; pending хранится отдельно (revision/draft), чтобы мир видел последний approve (SC-PACK-13).
- Alternative (одна строка pack = текущий draft) — отвергнута: ломает «мир видит approve».

### D3: Identity gates

| Действие | Кто |
|----------|-----|
| Catalog list / pack public view | JWT (incl. anonymous); blocked packs всё ещё видны с флагом |
| Add to collection | JWT (incl. anonymous, unverified) |
| Create / edit / submit | JWT + not anonymous + `emailVerified` |
| Staff queue / approve / reject / cancel / block / unblock | `moderator` \| `admin` (reuse `ht_role`) |

- Client: при ineligible create/edit — модалка login / confirm email (ссылка в кабинет / confirm flow), не silent 403 only.
- Server — источник истины на каждом mutating endpoint.

### D4: Submit validation

- ≥2 answer cards, ≥2 tasks, каждый task ≥1 filled slot, difficulty ∈ {1,2,3}.
- При изменении content referenced answer card — сервер (и editor UX) **очищает** слоты, ссылавшиеся на неё; submit 4xx пока есть пустые слоты.
- Один активный `pending` request на pack; повторный submit от того же change author обновляет draft того же request.

### D5: Lock + race

- `pending` ⇒ start-edit для других → 409/403.
- Change author может PATCH draft + resubmit.
- Второй submit от другого пользователя → ошибка; его draft сохранять per (user, pack) пока request не approved/cancelled.
- Staff **cancel** снимает pending (D11 explore).

### D6: Moderation thread + mail

- Сообщения только change author ↔ staff (как support thread visibility).
- После approve: новый цикл = новый request + новый thread.
- Reject: тот же thread; автор правит и resubmit.
- Mail: reuse `sendEmail` / smtp.bz; события approve, reject, staff message, block; skip anonymous; SPA hash links (`CLIENT_APP_URL`).

### D7: Co-author labels

- После approve вклада — подпись на task set (display). Права = только коллекция + verify gate. Нет отдельной ACL.

### D8: Block

- Flag `blocked` на pack; UI везде «заблокирован»; mutate edit/submit deny; catalog всё ещё показывает с бейджем. Unblock — staff only. Hard delete — out of scope.

### D9: HTTP surface (черновой контракт)

Префикс `/api/content/packs` (точные пути при apply; смысл):

**Public / user:** list catalog; get pack live; collection list/add/remove; create pack; get/put draft (if allowed); submit; get own moderation thread + post message.

**Staff:** list pending; get preview (submitted snapshot); approve; reject; cancel; post staff message; block; unblock.

Ошибки: 401/403/409 + тело для client banner.

### D10: Client structure

- Pages: catalog, collection, pack view, pack editor, staff moderation list + detail/thread.
- Store Pinia `content` (или `packs`) — весь HTTP I/O.
- Router: `requiresAuth` для раздела; staff routes дополнительно по `role`.
- i18n RU product copy; forms q-form + store error + q-banner.
- Slot UX: +/− slots; click card fills next empty; click slot clears (SC-PACK-07 semantics mirrored locally, server enforces).
- Nav entry from lobby/header (конкретная точка — apply + `work-with-pages`).

### D11: Skills / AGENTS

- После кода: точечно client/server AGENTS Business Entities + skills routes/pages/database/auth email gate; meta index если появится `content`.

### D12: Prerequisites from explore (закрыты продуктом)

Все D1–D12 explore закрыты в proposal/spec; технических внешних сервисов новых нет (mail/roles уже в проекте). Новые npm-зависимости не требуются.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Сложная модель live vs draft | Явные revision/snapshot таблицы; публичные GET только live |
| Гонки edit / «зависший» pending | Staff cancel; 409 на race; один pending на pack |
| Объём UI | Вертикальный HTTP+CRUD сначала; polish UX отдельными tasks |
| Путаница с peek rewards | Spec SC-PACK-33; difficulty только persist |
| Раздувание SQLite | Text-only; лимиты длины — разумные defaults при apply |

## Migration Plan

- Deploy server (ensure tables) → client.
- Пустой каталог до первых approve — ок.
- Rollback: feature routes можно отключить; таблицы оставлять.

## Open Questions

- Точные лимиты длины title/content/question (defaults при apply: ~120 / ~2k / ~2k) — не меняют поведение SC.
- Нужен ли remove-from-collection в v1 UI — да, симметрично add (предположение; не ломает spec).

Чеклист реализации — `tasks.md`.
