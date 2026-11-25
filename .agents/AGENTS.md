# Индекс агента — happy-tourist-meta

Канонический каталог документации и skills для AI-агентов в метарепозитории Happy Tourist (настольная игра «Счастливый турист»).

Краткие always-on инструкции (Cursor): [`AGENTS.md`](../AGENTS.md) в корне репозитория.

Runtime-код: siblings  
[`../happy-tourist.github.io`](../happy-tourist.github.io) (Vue 3 + Quasar SPA) и  
[`../happy-tourist-server`](../happy-tourist-server) (Colyseus).  
Контекст пакетов:

| Пакет | AGENTS.md |
|-------|-----------|
| Client | [`../happy-tourist.github.io/AGENTS.md`](../happy-tourist.github.io/AGENTS.md) |
| Server | [`../happy-tourist-server/AGENTS.md`](../happy-tourist-server/AGENTS.md) |

---

## Документация

| Документ | Назначение |
|----------|------------|
| [`docs/projects-map.md`](../docs/projects-map.md) | Сервис → sibling-путь; local YAML; Child `project-map.md` (ключ `happy-tourist-meta`) |
| [`docs/README.md`](../docs/README.md) | Оглавление документации |

Локальные оверрайды путей: [`projects-map.local.yaml`](../projects-map.local.yaml) / шаблон [`projects-map.local.yaml.example`](../projects-map.local.yaml.example) в корне meta.

OpenSpec: [`openspec/config.yaml`](../openspec/config.yaml); артефакты в `openspec/changes/` и `openspec/specs/`.

---

## Skills

### OpenSpec / workspace (`.agents/skills/`)

| Skill | Когда |
|-------|--------|
| [`align-code`](skills/align-code/SKILL.md) | Client + server: `*-align-code` + `*-verify-code` → сводный отчёт (петли + async-гонки) |
| [`align-meta`](skills/align-meta/SKILL.md) | Качество созданных/изменённых meta-артефактов (OpenSpec/docs/skills/indexes); paste-ready review cards; read-only |
| [`verify-mock`](skills/verify-mock/SKILL.md) | Client UI vs макет + **строгие размеры и цвета (hex/rgba / icon ink)** Visual Spec → Violations / Warnings / Recommendations |
| [`prepare-mock`](skills/prepare-mock/SKILL.md) | Снять с макета размеры/отступы/**hex·rgba цвета + icon ink**/copy/**все** dividers/колонки/высоту кнопок + **список недостающих иконок** → Visual Spec в design (+ specs); до propose/apply |
| [`commit`](skills/commit/SKILL.md) | Status → stage → commit → **push** по meta + client + server (main/master) |
| [`check-changes`](skills/check-changes/SKILL.md) | Client/server diffs (default unstaged; optional full branch) → Hard/Warn/Rec с add/change/delete/**split** (bloat ≥5 → обязательный split) + maps/docs/AGENTS/`openspec/config.yaml` |
| [`sync-changes`](skills/sync-changes/SKILL.md) | Двусторонний gap-анализ с другим meta: deep orchestration/OpenSpec + discovery (остальные skills, роли package-skills, indexes/docs/kilo) → что перенять сюда и туда |
| [`prepare-changes`](skills/prepare-changes/SKILL.md) | До propose: чаще всего **один** change (норма); дробить только по нужде; пункты работ + проверяемый результат; ⛔ вне стека → изучить инструмент; skills + код |
| [`review-change`](skills/review-change/SKILL.md) | Active change: brief-server → brief-client → align-back → align-front → align-meta (субагенты + «Добавь и поправь…»); resume по state; без propose/apply |
| [`implement-change`](skills/implement-change/SKILL.md) | Активный change: apply → **align** (`*-align-code`) → **verify** (`*-verify-code`) → verify-mock (**если** макет прикреплён и change про макет) → check-changes → commit; resume — state; standalone `align-code` не меняется |
| [`end-implement-change`](skills/end-implement-change/SKILL.md) | Закрытие change: update → sync-specs → archive → commit (без вопросов) |
| [`openspec-propose`](skills/openspec-propose/SKILL.md) | Propose: proposal → specs → design → tasks; при ровно одном active change — всегда дописывать в него (новый каталог только по явной просьбе) |
| [`openspec-new-change`](skills/openspec-new-change/SKILL.md) | Создать change и начать артефакты по схеме |
| [`openspec-continue-change`](skills/openspec-continue-change/SKILL.md) | Продолжить создание недостающих артефактов change |
| [`openspec-update-change`](skills/openspec-update-change/SKILL.md) | Правка существующих артефактов change без кода |
| [`openspec-apply-change`](skills/openspec-apply-change/SKILL.md) | Реализация по `tasks.md` |
| [`openspec-verify-change`](skills/openspec-verify-change/SKILL.md) | Проверка реализации против артефактов |
| [`openspec-archive-change`](skills/openspec-archive-change/SKILL.md) | Архивация закрытого change |
| [`openspec-explore`](skills/openspec-explore/SKILL.md) | Explore mode до/во время change; явно флагает «⛔ нужно решение разработчика» если фича ещё не implementable в проекте |
| [`openspec-sync-specs`](skills/openspec-sync-specs/SKILL.md) | Delta → main specs без archive |

### Discovery (другие корни)

| Корень | Назначение |
|--------|------------|
| [`.agents/skills/client/`](skills/client/) | Client-wide skills (meta); runtime: `../happy-tourist.github.io/` |
| [`.agents/skills/server/`](skills/server/) | Server-wide skills (meta); runtime: `../happy-tourist-server/` |

#### Client-wide skills (`.agents/skills/client/`)

Стек: Vue 3 Composition API / `<script setup>`, Quasar 2, Pinia 4, vue-router 5 (hash), vue-i18n 11, `@colyseus/sdk` 0.18, TypeScript. Runtime: `../happy-tourist.github.io/`.

| Skill | Когда |
|-------|--------|
| [`brief-client-scope`](skills/client/brief-client-scope/SKILL.md) | Digest постановки на client (UI/поля/binding/reuse + карточки gaps); read-only |
| [`align-implementability-front`](skills/client/align-implementability-front/SKILL.md) | Meta client claims vs `happy-tourist.github.io` (реализуемость) + `[be-without-fe]` + полнота Visual Spec при mock-driven; read-only |
| [`client-align-code`](skills/client/client-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов; реактивные/async-петли; async-гонки (await до `onStateChange` / первый `ROOM_STATE`); Quasar nested slots/overlays (presence rings ↔ avatar) |
| [`client-locate-change-points`](skills/client/client-locate-change-points/SKILL.md) | Где править (unified packs + favorites + author re-edit + staff take; CSV/tiles SC-PACK-210…255; portrait `PackAnswerCardTile` SC-PACK-222/225/282/283 + `PackTaskTile`/`taskFilledSlots` SC-PACK-278/283; compose dialogs SC-PACK-256…263/276…281 + peek mini-card pool; never-live draft + persisted lifecycle/`canHardDelete`/hidden shell SC-PACK-234/264…275; staff boot SC-PACK-231…238; maps SC-MAP-53…68; Vitest AnswerCompose/TaskCompose/AddTaskSetDraft/DraftLifecycle) |
| [`client-verify-code`](skills/client/client-verify-code/SKILL.md) | Проверка кода / compliance skills + DRY/KISS/YAGNI + Template→computed / `<script setup>` order (incl. sticky `.game-hud` presence + translucent α 0.78 / SC-PRESENCE-12/34 + sibling avatar) |
| [`client-work-with-review`](skills/client/client-work-with-review/SKILL.md) | Bugs / regressions / performance / UI overflow review (не verify/align) |
| [`colyseus-client`](skills/client/colyseus-client/SKILL.md) | `client.http` + room messages (`move`/`rescue`/`returnFromFinish`/`peekPlace`/`peekSubmit`/`say` + private `knowledge` with `outcome?`; `allJailWarning` inert; `allJailSkip` Notify; push/peek-open/endTurn retired; topic [game-messages.md](skills/client/colyseus-client/game-messages.md); create `{ mapId, packId, taskSetIds, maxSeats, opponentMode }` no density; seat `isBot`; not axios/BFF) |
| [`client-work-with-auth`](skills/client/client-work-with-auth/SKILL.md) | Colyseus Auth (email/anonymous/Google), password policy + zxcvbn meter, SPA confirm/reset + JSON, cabinet, userdata `role` / `isStaff` / `isAdmin`, verify reminder, `onChange`; router silent-guest / Login upgrade → topic [guards.md](skills/client/client-work-with-auth/guards.md); pack create/edit verify modal (lobby/game stay soft) |
| [`client-work-with-errors`](skills/client/client-work-with-errors/SKILL.md) | Store `error` + `q-banner` (pages + App theme; auth/game/support/content/maps), room `onError`; soft-drop rejoin gated by store `consentedLeaving` |
| [`client-work-with-structure`](skills/client/client-work-with-structure/SKILL.md) | pages/components/stores/boot/lib/assets + anti-passthrough wrapper gate (unified packs; `packs/pack-bg-*.jpg` + `app.scss` shell bg SC-BRAND-21 / grid underlay SC-PACK-285; `board/*.png` illustrated GamePage affordances; portrait `PackAnswerCardTile` SC-PACK-282/283 + `PackTaskTile`/`taskFilledSlots` SC-PACK-278/283; compose SC-PACK-256…263/276…281; AddTaskSet quiet autosave + top delete by `canHardDelete` SC-PACK-264…266/274/275; CSV/tiles SC-PACK-210…255; no collection-first; soft-unpublish; cascade) |
| [`work-with-forms`](skills/client/work-with-forms/SKILL.md) | LoginPage / ForgotPasswordPage / ResetPasswordPage / AccountPage (icon-only logout SC-BRAND-32) / Support create+reply `q-form` (incl. `change_pack` **in-catalog** pack select; content moderation reply Editor/AddTaskSet; policy meter on register/reset/change; `lazy-rules` + clear → `nextTick` → `resetValidation`) / guest + Google buttons |
| [`work-with-pages`](skills/client/work-with-pages/SKILL.md) | Routes + pages; topics: [shell.md](skills/client/work-with-pages/shell.md), [content-pages.md](skills/client/work-with-pages/content-pages.md), [game-page.md](skills/client/work-with-pages/game-page.md) (strip knowledge + lightbulb + 60s ring + Notify + finish\|cut modal; no density/end-turn/push/all-jail modal) |
| [`work-with-stores`](skills/client/work-with-stores/SKILL.md) | Pinia domains; topics: [content.md](skills/client/work-with-stores/content.md), [pack-lifecycle.md](skills/client/work-with-stores/pack-lifecycle.md) (`hasUnsubmittedChanges`/`canHardDelete` SC-PACK-234/264…275), [maps.md](skills/client/work-with-stores/maps.md) (palette/`MapGridPreview` shared `tileSkins` SC-MAP-54/71/72), [game.md](skills/client/work-with-stores/game.md) (`opponentMode` + seat `isBot`) |
| [`work-with-styles`](skills/client/work-with-styles/SKILL.md) | Quasar Dark / variables / utilities; topics: [theme.md](skills/client/work-with-styles/theme.md), [board.md](skills/client/work-with-styles/board.md) (knowledge + lightbulb; strip TR yellow only; red TR above grille; focus@3 + rescue/return; no board yellow knowledge / end-turn/peek/push CSS; HUD α 0.78), [pack-cards.md](skills/client/work-with-styles/pack-cards.md) |
| [`work-with-localization`](skills/client/work-with-localization/SKILL.md) | vue-i18n (`en-US` catalog, RU copy); namespaces `auth`/`header`/`lobby`/`game`/`support`/`content`/`maps` — key details in body / store topics |
| [`work-with-lobby`](skills/client/work-with-lobby/SKILL.md) | Live LobbyRoom subscribe; create map+pack+taskSets + maxSeats + opponentMode people\|ai (`person`/`smart_toy` — SC-LOBBY-50…52; **no** density — SC-LOBBY-40/41); map mini preview SC-LOBBY-31; listing **`LobbyRoomCardTile`** + `taskCount` — topic [listing-cards.md](skills/client/work-with-lobby/listing-cards.md); after tourist attach **`void unsubscribeLobby()`** (SC-LOBBY-05/37); quiet resubscribe; section nav in App header — not LobbyPage |
| [`work-with-rooms`](skills/client/work-with-rooms/SKILL.md) | Room lifecycle (`_enterRoom` → attach → `void unsubscribeLobby`); tourist reconnect token; consented leave (logo + `header-game-leave` confirm in App on Game) |
| [`work-with-game-board`](skills/client/work-with-game-board/SKILL.md) | Board from synced `grid` + stable `boardPieces` + presence HUD (say@12 + focus@3 + 60s ring; knowledge counter + Material `lightbulb`; strip TR yellow knowledge; red TR `remainingTurns` above grille; bot `isBot` «AI» caption SC-PRESENCE-60 — topics `knowledge.md`/`peek.md`/`focus.md`) + open-on-land Q&A + Notify peek/`allJailSkip` + grille/catapult anim + free rescue/return (**no** push/end-turn/peek-open) + finish\|cut modal + travel clip (`lib/travelClip`); brand-logo leave in App |
| [`work-with-env-deploy`](skills/client/work-with-env-deploy/SKILL.md) | `VITE_*`, hash router, GitHub Pages |
| [`client-work-with-test`](skills/client/client-work-with-test/SKILL.md) | Vitest; catalog/lobby chrome + HUD α0.78 + GameBoardWire + `travelClip` / GameFinishTravelClip + GamePresenceFocus **focus@3** / GamePresenceAi caption + Material lightbulb / turn ring + Notify peek/`allJailSkip` + finish\|cut + LobbyCreate opponentMode + **no density** + `game.peekStatusFlash` + lobby hang leave SC-LOBBY-05/37; leftover ∞ / end-turn / push / density / all-jail modal / `mdi-lightbulb` cases retired; details in topic `stores.md` / `composables.md` / `colyseus.md` |

#### Server-wide skills (`.agents/skills/server/`)

Стек: Colyseus 0.18, `@colyseus/tools`, `@colyseus/auth`, `@colyseus/schema`, Express 5, Drizzle + SQLite, mocha + `@colyseus/testing`. Runtime: `../happy-tourist-server/`.

| Skill | Когда |
|-------|--------|
| [`brief-server-scope`](skills/server/brief-server-scope/SKILL.md) | Digest постановки на server (контракт/поведение/reuse + карточки gaps); read-only |
| [`align-implementability-back`](skills/server/align-implementability-back/SKILL.md) | Meta server claims vs `happy-tourist-server` (реализуемость) + `[fe-only-without-be]`; read-only |
| [`server-align-code`](skills/server/server-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов; handler/lifecycle-петли; first-sync races |
| [`server-locate-change-points`](skills/server/server-locate-change-points/SKILL.md) | Где править (content unified list/favorites/author re-edit/staff take/`neverLive`/`openRequestType`; never-live draft + persisted lifecycle/`canHardDelete`/hidden shell D9–D14 SC-PACK-234/264…275; staff SC-PACK-230/233; `taskSetsPreview` SC-PACK-252/253; maps SC-MAP-64; grants retired SC-PACK-170) |
| [`server-verify-code`](skills/server/server-verify-code/SKILL.md) | Проверка кода на соответствие skills + DRY/KISS/YAGNI |
| [`server-work-with-auth`](skills/server/server-work-with-auth/SKILL.md) | `@colyseus/auth`, Google, mailer, password policy, SPA confirm/reset, `emailVerified`, `htRole`, `BOOTSTRAP_ADMIN_IDS`, `DEFAULT_CONTENT_PACK_IDS` grant **no-op** (SC-PACK-170), JWT `onAuth` |
| [`server-work-with-errors`](skills/server/server-work-with-errors/SKILL.md) | Auth/game failures, HTTP health, client-facing errors |
| [`server-work-with-structure`](skills/server/server-work-with-structure/SKILL.md) | rooms/schema/db/config/lib (content unified list + `neverLive` + draft put/discard/`retainedNeverLive*` + persisted dirty/`canHardDelete`/hidden shell D9–D14 SC-PACK-264…275 + staff SC-PACK-230/233 + `taskSetsPreview` SC-PACK-252/253; maps SC-MAP-64; grants no-op; support/mailer) |
| [`server-work-with-test`](skills/server/server-work-with-test/SKILL.md) | mocha; content packs/maps as before; rooms knowledge economy (private `knowledge`+`outcome?`; open-on-land; exclusive 15%/20%; `remainingTurns`; push/endTurn/peek-open reject; multi 60s `turnUntil`; no leftover ∞/`timeExpired`; `allJailSkip` SC-MOVE-124/125; `peekClose.reason` wrong|timeout; trap presentation 1500/1200; Easy AI/`botEasy` SC-AI-*) / auth/support |
| [`work-with-rooms`](skills/server/work-with-rooms/SKILL.md) | Room lifecycle (`onDrop`/`onReconnect`/`onLeave` leave-clear holding / `onDispose` cancel trap pipeline), registration, create via `roomContentSnapshot` (mapId/packId/taskSetIds + maxSeats + opponentMode; density ignored; effective AI private+bots) / waiting-only seating / phase / seats; listing metadata `taskSetLabels[+taskCount]` SC-LOBBY-32 |
| [`work-with-schema`](skills/server/work-with-schema/SKILL.md) | `@colyseus/schema` sync (`phase` / `maxSeats` / `grid` / peek session + seats pieces by pieceId incl. `remainingTurns` + `finishPlace` 0|1 + `isBot` + `turnUntil`/`turnBudgetSeconds` + `removedTaskKeys` + grilles/catapults; knowledge/`allJailWarning` not schema) |
| [`work-with-messages`](skills/server/work-with-messages/SKILL.md) | Room `onMessage('move'|'rescue'|'returnFromFinish'|'peekPlace'|'peekSubmit'|'ready'|'say')` + private `knowledge` (`outcome?`); `allJailWarning` retired; `allJailSkip` when eligible ≥2; push/peek-open/endTurn reject; open-on-land; multi timeout `peekClose`; paced trap pipeline |
| [`work-with-game`](skills/server/work-with-game/SKILL.md) | Waiting-only seating + deferred pieces + content snapshot + BoardGeometry + private knowledge + open-on-land peek + one move|return + free rescue + exclusive traps 15%/20% + paced trap pipeline + `remainingTurns` + 60s multi deadline + all-jail wait/skip + `allJailSkip` + penultimate cut + Easy AI (`botEasy` / `Seat.isBot` / sessionId helpers); push/endTurn/solo leftover lock/density/all-jail teleport retired; `touristMove.ts`; room `tourist` |
| [`work-with-routes`](skills/server/work-with-routes/SKILL.md) | HTTP thin routes; topic [content.md](skills/server/work-with-routes/content.md) for `/api/content/*` (draft put/get/discard + dirty/`canHardDelete`/hidden shell SC-PACK-264…275 + maps/staff); auth/theme/support/admin in core |
| [`work-with-middleware`](skills/server/work-with-middleware/SKILL.md) | CORS, `/health`, monitor/playground |
| [`work-with-config`](skills/server/work-with-config/SKILL.md) | env, secrets, OAuth + email `config/auth`, `BOOTSTRAP_ADMIN_IDS`, `DEFAULT_CONTENT_PACK_IDS` (grants retired), `defineServer` |
| [`work-with-database`](skills/server/work-with-database/SKILL.md) | GameDatabase + users + support + content_* (favorites; moderation take; `in_catalog`; `draft_activated`/`has_unsubmitted_changes`; grants retired) + content_maps |
| [`work-with-env-deploy`](skills/server/work-with-env-deploy/SKILL.md) | VPS, PM2, GitHub Actions rsync; smtp.bz + `AUTH_BACKEND_URL` / `CLIENT_APP_URL` / `BOOTSTRAP_ADMIN_IDS` / `DEFAULT_CONTENT_PACK_IDS` |
| [`work-with-loadtest`](skills/server/work-with-loadtest/SKILL.md) | `@colyseus/loadtest` scripts |

Runtime-пути в skills (`src/…`) — относительно корня соответствующего sibling-репозитория.

Kilo Code: канон [`../kilo.template.jsonc`](../kilo.template.jsonc); локально `cp kilo.template.jsonc kilo.jsonc` (ignored). Nested skills — `skills.paths` в рабочем `kilo.jsonc` (`.agents/skills/client`, `.agents/skills/server`).

---

## OpenSpec / workflow

- Конфиг: [`openspec/config.yaml`](../openspec/config.yaml) (`projectsMap`, `context`, capability rules).
- Capability ID = продуктовый путь (auth/login, lobby/rooms, game/move, content/packs, …), не имя change.
- Delta: `openspec/changes/<slug>/specs/<capability-id>/spec.md`
- Main: `openspec/specs/<capability-id>/spec.md`
- Типичный Cursor chat workflow: `/opsx-explore` → [`prepare-changes`](skills/prepare-changes/SKILL.md) (если scope широкий) → `/opsx-propose` → [`review-change`](skills/review-change/SKILL.md) (опционально: brief + align постановки) → review → `/opsx-apply` / [`implement-change`](skills/implement-change/SKILL.md) → `/opsx-sync` → `/opsx-archive`.
- Артефакты живут в **happy-tourist-meta**, не в siblings. Пути к runtime при apply — через [`docs/projects-map.md`](../docs/projects-map.md) (+ local YAML).
- Always-on: [`AGENTS.md`](../AGENTS.md).
