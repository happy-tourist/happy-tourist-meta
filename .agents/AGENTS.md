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
| [`align-implementability-back`](skills/align-implementability-back/SKILL.md) | Meta server claims vs `happy-tourist-server` (реализуемость) + `[fe-only-without-be]`; read-only |
| [`align-implementability-front`](skills/align-implementability-front/SKILL.md) | Meta client claims vs `happy-tourist.github.io` (реализуемость) + `[be-without-fe]` + полнота Visual Spec (размеры/отступы/цвета) при mock-driven; read-only |
| [`align-meta`](skills/align-meta/SKILL.md) | Качество созданных/изменённых meta-артефактов (OpenSpec/docs/skills/indexes); read-only |
| [`verify-mock`](skills/verify-mock/SKILL.md) | Client UI vs макет + **строгие размеры и цвета (hex/rgba / icon ink)** Visual Spec → Violations / Warnings / Recommendations |
| [`prepare-mock`](skills/prepare-mock/SKILL.md) | Снять с макета размеры/отступы/**hex·rgba цвета + icon ink**/copy/**все** dividers/колонки/высоту кнопок + **список недостающих иконок** → Visual Spec в design (+ specs); до propose/apply |
| [`commit`](skills/commit/SKILL.md) | Status → stage → commit → **push** по meta + client + server (main/master) |
| [`check-changes`](skills/check-changes/SKILL.md) | Unstaged client/server → add/change/delete/**split** (скоринг bloat ≥5 → обязательный план topic/sibling; запрет description-dump) + maps/docs/AGENTS |
| [`prepare-changes`](skills/prepare-changes/SKILL.md) | До propose: чаще всего **один** change (норма); дробить только по нужде; пункты работ + проверяемый результат; ⛔ вне стека → изучить инструмент; skills + код |
| [`align-change`](skills/align-change/SKILL.md) | Active change: align-back → align-front → align-meta (субагенты + «Добавь и поправь…»); resume по state; без propose/apply |
| [`implement-change`](skills/implement-change/SKILL.md) | Активный change: apply → align → verify-mock (**если** макет прикреплён и change про макет) → check-changes → commit; resume — повторный запуск (state) |
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
| [`client-align-code`](skills/client/client-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов; реактивные/async-петли; async-гонки (await до `onStateChange` / первый `ROOM_STATE`); Quasar nested slots/overlays (presence rings ↔ avatar) |
| [`client-locate-change-points`](skills/client/client-locate-change-points/SKILL.md) | Где править (unified packs + favorites + author re-edit + staff take; answers/tasks CSV + playing-card + catalog `PackListCardTile` ~180×260 + published-only set preview / task-set `PackTaskSetCardTile` `packContentCsv`/framed CSV/`PackTasksCsvControls`/`PackAnswerCardTile`/`PackTaskTile`/`MapListCardTile` SC-PACK-210…229/239…254; staff boot/dirty Submit/peek slots/`editorDirty` SC-PACK-231…238; maps SC-MAP-53…56 / 62…65; Lobby crumb SC-BRAND-20 + selected mini SC-LOBBY-31; Vitest ContentUnifiedList/AuthorEditTake/Maps/CSV/tiles/list-cards/DirtySubmit/StaffBoot) |
| [`client-verify-code`](skills/client/client-verify-code/SKILL.md) | Проверка кода / compliance skills + DRY/KISS/YAGNI (incl. sticky `.game-hud` presence + sibling avatar / SC-PRESENCE-12) |
| [`colyseus-client`](skills/client/colyseus-client/SKILL.md) | `client.http` + room messages (`move`/`rescue`/`push`/`returnFromFinish`/`peek`/`peekPlace`/`peekSubmit`/`endTurn`/`say` + private `budgets`/`allJailWarning`; create `{ mapId, packId, taskSetIds, … }`; synced peek session; not axios/BFF) |
| [`client-work-with-auth`](skills/client/client-work-with-auth/SKILL.md) | Colyseus Auth (email/anonymous/Google), password policy + zxcvbn meter, SPA confirm/reset + JSON, cabinet displayName/change-password/`emailVerified`, userdata `role` / `isStaff` / `isAdmin`, once-per-session verify reminder, `onChange`, route guards (`requiresStaff`/`requiresAdmin`); pack create/edit verify modal (lobby/game stay soft) |
| [`client-work-with-errors`](skills/client/client-work-with-errors/SKILL.md) | Store `error` + `q-banner` (pages + App theme; auth/game/support/content/maps), room `onError`; soft-drop rejoin gated by store `consentedLeaving` |
| [`client-work-with-structure`](skills/client/client-work-with-structure/SKILL.md) | pages/components/stores/boot/lib (unified packs + favorites + author re-edit + staff take; `packContentCsv` + `editorDirty` + framed CSV / `PackTasksCsvControls` / catalog `PackListCardTile` ~180×260 + published-only set preview / task-set `PackTaskSetCardTile` / `MapListCardTile` SC-PACK-210…229/239…254 / 231…238 / SC-MAP-55/62…65; no collection-first; no non-staff my-moderation nav; soft-unpublish; cascade) |
| [`work-with-forms`](skills/client/work-with-forms/SKILL.md) | LoginPage / ForgotPasswordPage / ResetPasswordPage / AccountPage / Support create+reply `q-form` (incl. `change_pack` **in-catalog** pack select; content moderation reply Editor/AddTaskSet; policy meter on register/reset/change; `lazy-rules` + clear → `nextTick` → `resetValidation`) / guest + Google buttons |
| [`work-with-pages`](skills/client/work-with-pages/SKILL.md) | Routes + pages; topics: [shell.md](skills/client/work-with-pages/shell.md) (App header/crumbs/leave), [content-pages.md](skills/client/work-with-pages/content-pages.md) (packs/maps UI), [game-page.md](skills/client/work-with-pages/game-page.md) |
| [`work-with-stores`](skills/client/work-with-stores/SKILL.md) | Pinia domains; topics: [content.md](skills/client/work-with-stores/content.md), [maps.md](skills/client/work-with-stores/maps.md), [game.md](skills/client/work-with-stores/game.md) |
| [`work-with-styles`](skills/client/work-with-styles/SKILL.md) | Quasar Dark / variables / utilities; topics: [theme.md](skills/client/work-with-styles/theme.md), [board.md](skills/client/work-with-styles/board.md), [pack-cards.md](skills/client/work-with-styles/pack-cards.md) (catalog `PackListCardTile` SC-PACK-228/249…254 + `PackTaskSetCardTile` SC-PACK-229/239…248) |
| [`work-with-localization`](skills/client/work-with-localization/SKILL.md) | vue-i18n (`en-US` catalog, RU copy); namespaces `auth`/`header`/`lobby`/`game`/`support`/`content`/`maps` — key details in body / store topics |
| [`work-with-lobby`](skills/client/work-with-lobby/SKILL.md) | Live LobbyRoom subscribe; create map+pack+taskSets + chosen maxSeats (1…map.players) + densities; map option + **selected-item** mini `MapGridPreview` SC-LOBBY-31 + pack/set ordinals **without** set author (`taskSetOption` / `packSetsCaption` SC-LOBBY-24/26); join busy-lock; quiet resubscribe; Packs/Maps/Support/Модерация/logout in App header — not LobbyPage |
| [`work-with-rooms`](skills/client/work-with-rooms/SKILL.md) | Room lifecycle, tourist reconnect token, consented leave (logo + `header-game-leave` confirm in App on Game) |
| [`work-with-game-board`](skills/client/work-with-game-board/SKILL.md) | Board from synced `grid` (`lib/boardGeometry`) + pieceId pieces + presence HUD + focus affordance + shared peek Q&A (topic `peek.md`/`focus.md`) + grille/catapult anim + trap/rescue/push/return + say; brand-logo leave/status in App |
| [`work-with-env-deploy`](skills/client/work-with-env-deploy/SKILL.md) | `VITE_*`, hash router, GitHub Pages |
| [`work-with-test`](skills/client/work-with-test/SKILL.md) | Vitest; ContentUnifiedList/`openRequestType` + AuthorEditTake/`neverLive` + pack CSV + tiles/list-cards SC-PACK-210…229/239…254 (`packContentCsv`/framed CSV/`PackTasksCsvControls`/`ContentPackEditorCsv`/`PackAnswerCardTile`/`PackTaskTile`/`PackListCardTile`/`PackTaskSetCardTile`/`MapListCardTile`/`ContentPackPlayingCards`/`ContentPackListCards`) + staff boot/dirty Submit/CSV hide/peek slots SC-PACK-231…238 + maps SC-MAP-41…56 / 62…65 + AppHeaderChrome + LobbyCreateWire ordinals SC-LOBBY-24/26; details in topic `stores.md` |

#### Server-wide skills (`.agents/skills/server/`)

Стек: Colyseus 0.18, `@colyseus/tools`, `@colyseus/auth`, `@colyseus/schema`, Express 5, Drizzle + SQLite, mocha + `@colyseus/testing`. Runtime: `../happy-tourist-server/`.

| Skill | Когда |
|-------|--------|
| [`server-align-code`](skills/server/server-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов; handler/lifecycle-петли; first-sync races |
| [`server-locate-change-points`](skills/server/server-locate-change-points/SKILL.md) | Где править (content unified list/favorites/author re-edit/staff take/cancel→draft/set marks without fan-out/`neverLive`/`openRequestType`/D22 cancel restore SC-PACK-196/197; staff false-draft omit SC-PACK-230 / always clear working after live staff-save SC-PACK-233; `listCatalog` `taskSetsPreview` SC-PACK-252/253; maps statuses + SC-MAP-64; default grants retired SC-PACK-170) |
| [`server-verify-code`](skills/server/server-verify-code/SKILL.md) | Проверка кода на соответствие skills + DRY/KISS/YAGNI |
| [`server-work-with-auth`](skills/server/server-work-with-auth/SKILL.md) | `@colyseus/auth`, Google, mailer, password policy, SPA confirm/reset, `emailVerified`, `htRole`, `BOOTSTRAP_ADMIN_IDS`, `DEFAULT_CONTENT_PACK_IDS` grant **no-op** (SC-PACK-170), JWT `onAuth` |
| [`server-work-with-errors`](skills/server/server-work-with-errors/SKILL.md) | Auth/game failures, HTTP health, client-facing errors |
| [`server-work-with-structure`](skills/server/server-work-with-structure/SKILL.md) | rooms/schema/db/config/lib (content unified list + favorites + author re-edit + staff take + working≠live draft + set marks without fan-out + `neverLive` + cancel restore D22 + `openRequestType` + `listCatalog` `taskSetsPreview` SC-PACK-252/253 + staff SC-PACK-230/233 always clear working; maps SC-MAP-64; default grants no-op; support/mailer) |
| [`server-work-with-test`](skills/server/server-work-with-test/SKILL.md) | mocha; zz-contentPacks unify/favorites/author-edit/take/cancel→draft/set marks + fan-out fix/`neverLive` SC-PACK-187…189 + cancel restore SC-PACK-196/197 + staff false-draft SC-PACK-230 + clear hanging working SC-PACK-233 + list `taskSetsPreview` SC-PACK-252/253; zz-contentMaps SC-MAP-64; zz-defaultContentPacks SC-PACK-170; rooms/auth/support |
| [`work-with-rooms`](skills/server/work-with-rooms/SKILL.md) | Room lifecycle (`onDrop`/`onReconnect`/`onLeave` leave-clear holding / `onDispose` clearTurnDeadline + cancel trap pipeline), registration, create via `roomContentSnapshot` (mapId/packId/taskSetIds + chosen maxSeats ≤ map.players) + grille/catapult density / waiting-only seating / phase / seats |
| [`work-with-schema`](skills/server/work-with-schema/SKILL.md) | `@colyseus/schema` sync (`phase` / `maxSeats` / `grid` / `touristsPerPlayer` / peek session / `flippedCells` / `answerCards` + seats pieces by pieceId + turn + `removedTaskKeys` + grilles/catapults) |
| [`work-with-messages`](skills/server/work-with-messages/SKILL.md) | Room `onMessage('move'|'rescue'|'push'|'returnFromFinish'|'peek'|'peekPlace'|'peekSubmit'|'endTurn'|'ready'|'say')` + private `budgets`/`allJailWarning` (pieceId payloads; shared peek; paced trap pipeline; peeks∞) |
| [`work-with-game`](skills/server/work-with-game/SKILL.md) | Waiting-only seating + deferred pieces by pieceId + content snapshot + BoardGeometry + shared peek deck + start/ready/countdown + reconnect + budgets + grille/catapult density + paced trap pipeline + rescue/push/return + `touristMove.ts`; room `tourist` |
| [`work-with-routes`](skills/server/work-with-routes/SKILL.md) | HTTP thin routes; `/api/content/*` unified list (`openRequestType`; working≠live → draft; lightweight `taskSetsPreview` SC-PACK-252/253) + favorite + author draft/lock + set `moderationStatus` / staff omit false draft SC-PACK-230 + `neverLive` ghost + add-task-set cancel restore D22 + staff-save always clear working SC-PACK-233 / maps SC-MAP-64 + staff take/release; collection legacy; grants retired |
| [`work-with-middleware`](skills/server/work-with-middleware/SKILL.md) | CORS, `/health`, monitor/playground |
| [`work-with-config`](skills/server/work-with-config/SKILL.md) | env, secrets, OAuth + email `config/auth`, `BOOTSTRAP_ADMIN_IDS`, `DEFAULT_CONTENT_PACK_IDS` (grants retired), `defineServer` |
| [`work-with-database`](skills/server/work-with-database/SKILL.md) | GameDatabase + users + support + content_* (favorites; moderation take; `in_catalog`; grants retired) + content_maps |
| [`work-with-env-deploy`](skills/server/work-with-env-deploy/SKILL.md) | VPS, PM2, GitHub Actions rsync; smtp.bz + `AUTH_BACKEND_URL` / `CLIENT_APP_URL` / `BOOTSTRAP_ADMIN_IDS` / `DEFAULT_CONTENT_PACK_IDS` |
| [`work-with-loadtest`](skills/server/work-with-loadtest/SKILL.md) | `@colyseus/loadtest` scripts |

Runtime-пути в skills (`src/…`) — относительно корня соответствующего sibling-репозитория.

Kilo Code: при появлении канона [`../kilo.template.jsonc`](../kilo.template.jsonc); локально `cp kilo.template.jsonc kilo.jsonc` (ignored). Nested skills — `skills.paths` в рабочем `kilo.jsonc`.

---

## OpenSpec / workflow

- Конфиг: [`openspec/config.yaml`](../openspec/config.yaml) (`projectsMap`, `context`, capability rules).
- Capability ID = продуктовый путь (auth/login, lobby/rooms, game/move, content/packs, …), не имя change.
- Delta: `openspec/changes/<slug>/specs/<capability-id>/spec.md`
- Main: `openspec/specs/<capability-id>/spec.md`
- Типичный Cursor chat workflow: `/opsx-explore` → [`prepare-changes`](skills/prepare-changes/SKILL.md) (если scope широкий) → `/opsx-propose` → [`align-change`](skills/align-change/SKILL.md) (опционально) → review → `/opsx-apply` / [`implement-change`](skills/implement-change/SKILL.md) → `/opsx-sync` → `/opsx-archive`.
- Артефакты живут в **happy-tourist-meta**, не в siblings. Пути к runtime при apply — через [`docs/projects-map.md`](../docs/projects-map.md) (+ local YAML).
- Always-on: [`AGENTS.md`](../AGENTS.md).
