## Context

См. `proposal.md` — Why / Scope. Коллекция уже есть: `content_pack_collections` (PK `user_id, pack_id`), `addToCollection` требует live + `inCatalog` + not blocked. Паттерн one-shot boot: `backfillLegacyEmailVerified` + `db.configs`. `BOOTSTRAP_ADMIN_IDS` — только роли; **не** использовать для выбора пака.

Пакет: **server only**. Client без правок. Tourist-room / peek — out of scope.

## Goals / Non-Goals

**Goals:**

- Env CSV pack ids → одноразовая выдача in-catalog паков всем текущим и каждому новому user
- Sticky remove без таблицы opt-out (не вызывать grant на login / `GET /collection`)
- Пропуск невалидных id без падения процесса

**Non-Goals:**

- Ledger / opt-out table (достаточно one-shot call sites)
- Client UX, peek wiring
- Выбор паков по creator = admin

## Decisions

### D1 — Env `DEFAULT_CONTENT_PACK_IDS`

- CSV pack ids (trim, skip empty). Имя рядом с `BOOTSTRAP_ADMIN_IDS` в `.env.example` + строка в server `AGENTS.md`.
- Альтернатива «все паки createdBy admin» — отвергнута (explore D1=A).

### D2 — One-shot call sites (B′), без re-ensure

```
boot --> backfillAllUsers(eligiblePacks)   # once per pack id via db.configs key
create user --> grantDefaults(userId)      # register / Google / anonymous
GET /collection, login, onParseToken --> NEVER grant
```

- `INSERT` membership only if absent (`INSERT OR IGNORE` / exists-check) — идемпотентность backfill.
- Sticky remove: после DELETE строка исчезает; повторный grant **не** вызывается → пак не возвращается.
- Альтернатива «ensure на каждый GET /collection» — отвергнута (ломает D3).

### D3 — Per-pack backfill flag

- Ключ configs вроде `default_pack_backfill:<packId>` (или один blob со списком уже обработанных id).
- Новый id в env на следующем старте → one-shot backfill **этого** id всем существующим users; уже обработанные id не трогать снова.
- Альтернатива «один флаг на всю фичу» — хуже при добавлении второго default позже.

### D4 — Shared helper

- `parseDefaultContentPackIds()`, `listEligibleDefaultPacks()` (live + inCatalog + !blocked), `grantDefaultPacksToUser(userId)`, `backfillDefaultPacksOnBoot()`.
- Разместить в `src/lib/content.ts` (или тонкий `src/lib/defaultContentPacks.ts` + re-export); вызов boot из `app.config.ts` `express()` рядом с `ensureContentTables` / `bootstrapAdmins`.
- Create-time: wrap email register (как displayName), OAuth callback после создания user, **и** anonymous create path (Colyseus `signInAnonymously` → найти/обернуть точку появления строки в `colyseus_users`; mocha SC-PACK-143 обязан покрыть anonymous).

### D5 — Soft failures

- Неизвестный / unpublished id → log warn, skip; boot и register не 500.

### Prerequisites (explore)

| ID | Решение |
|----|---------|
| D1–D5 explore | закрыты в proposal |
| Ops | pack должен быть in-catalog до полезного grant; id в env после деплоя |

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Нет явного hook на anonymous create | Spike в apply; mocha на `signInAnonymously`; при отсутствии hook — узкий post-insert путь, **не** GET /collection |
| Большой backfill на старте | Один проход INSERT OR IGNORE; обычно мало users на VPS |
| Env указывает unpublished | Skip + warn; SC-PACK-144 |
| Второй pack id в env | Per-pack backfill key (D3) |

## Migration Plan

1. Deploy server с кодом + пустым `DEFAULT_CONTENT_PACK_IDS` (no-op).
2. Убедиться, что пробный пак in-catalog; прописать pack id в env; restart → backfill.
3. Rollback: очистить env и/или убрать код; membership в коллекциях остаются (не каскадить delete).

Чеклист реализации — `tasks.md`.
