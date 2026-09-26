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
| [`commit`](skills/commit/SKILL.md) | Status → stage → commit → **push** по meta + client + server (main/master) |
| [`check-changes`](skills/check-changes/SKILL.md) | Unstaged client/server → предложения: добавить/изменить/удалить/разбить (bloat → sibling или topic `*.md`) skills, maps, docs, AGENTS.md |
| [`prepare-changes`](skills/prepare-changes/SKILL.md) | До propose: чаще всего **один** change (норма); дробить только по нужде; пункты работ + проверяемый результат; ⛔ вне стека → изучить инструмент; skills + код |
| [`implement-change`](skills/implement-change/SKILL.md) | Активный change: apply (блоки tasks в субагентах) → align → check-changes → commit; resume — повторный запуск (state) |
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
| [`client-locate-change-points`](skills/client/client-locate-change-points/SKILL.md) | Где править (unified packs + favorites + author re-edit + staff take; maps filters; Vitest ContentUnifiedList/AuthorEditTake/Maps) |
| [`client-verify-code`](skills/client/client-verify-code/SKILL.md) | Проверка кода / compliance skills + DRY/KISS/YAGNI (incl. sticky `.game-hud` presence + sibling avatar / SC-PRESENCE-12) |
| [`colyseus-client`](skills/client/colyseus-client/SKILL.md) | `client.http` + room messages (`move`/`rescue`/`push`/`returnFromFinish`/`peek`/`peekPlace`/`peekSubmit`/`endTurn`/`say` + private `budgets`/`allJailWarning`; create `{ mapId, packId, taskSetIds, … }`; synced peek session; not axios/BFF) |
| [`client-work-with-auth`](skills/client/client-work-with-auth/SKILL.md) | Colyseus Auth (email/anonymous/Google), password policy + zxcvbn meter, SPA confirm/reset + JSON, cabinet displayName/change-password/`emailVerified`, userdata `role` / `isStaff` / `isAdmin`, once-per-session verify reminder, `onChange`, route guards (`requiresStaff`/`requiresAdmin`); pack create/edit verify modal (lobby/game stay soft) |
| [`client-work-with-errors`](skills/client/client-work-with-errors/SKILL.md) | Store `error` + `q-banner` (pages + App theme; auth/game/support/content/maps), room `onError`; soft-drop rejoin gated by store `consentedLeaving` |
| [`client-work-with-structure`](skills/client/client-work-with-structure/SKILL.md) | pages/components/stores/boot (unified packs + favorites + author re-edit + staff take; no collection-first; no non-staff my-moderation nav; soft-unpublish; cascade) |
| [`work-with-forms`](skills/client/work-with-forms/SKILL.md) | LoginPage / ForgotPasswordPage / ResetPasswordPage / AccountPage / Support create+reply `q-form` (incl. `change_pack` **in-catalog** pack select; content moderation reply Editor/AddTaskSet; policy meter on register/reset/change; `lazy-rules` + clear → `nextTick` → `resetValidation`) / guest + Google buttons |
| [`work-with-pages`](skills/client/work-with-pages/SKILL.md) | Routes + App shell (section nav / crumbs **inside** `q-page-container` SC-BRAND-17…18 / narrow burger fold Acc/Theme/Logout SC-BRAND-19 / session logout / Game leave); Lobby room-list only; unified packs filters/star/`openRequestType`; collection redirect; author re-edit; staff take; MapsList/MapEditor view vs paint + title-row Edit |
| [`work-with-stores`](skills/client/work-with-stores/SKILL.md) | Pinia domains; `content` unified list + `openRequestType` + cancel→draft + set marks / `neverLive` + add-task-set cancel restore D22 ([content.md](skills/client/work-with-stores/content.md)); `maps` statuses/author→Edit/title-row Edit/cancel→draft ([maps.md](skills/client/work-with-stores/maps.md)) |
| [`work-with-styles`](skills/client/work-with-styles/SKILL.md) | Quasar Dark + GET/POST `/api/theme`, header always-button brand-logo ≥60px CSS + theme toggle, `.app-breadcrumbs` inside `q-page-container` offset (SC-BRAND-17), muted chrome, tourist board (holes + grille overlays / `--grille-anim-ms` 1000) + catapult reveal/broken-hold CSS + top presence + seated strip HUD / budgets beside / end-turn icon / peek-rescue-push top-center / say top↓ own↑ CSS; content `.cascade-gap-outline` (SC-PACK-126) |
| [`work-with-localization`](skills/client/work-with-localization/SKILL.md) | vue-i18n; `header.*` section nav + `theme` (SC-BRAND-11…19); `content.*` / `maps.*` filters/statuses (published list rows: no badge); errors `author_request_open`/`moderation_taken`; support/auth/game |
| [`work-with-lobby`](skills/client/work-with-lobby/SKILL.md) | Live LobbyRoom subscribe; create map+pack+taskSets + chosen maxSeats (1…map.players) + densities; map preview + pack/set labels; join busy-lock; quiet resubscribe; Packs/Maps/Support/Модерация/logout in App header — not LobbyPage |
| [`work-with-rooms`](skills/client/work-with-rooms/SKILL.md) | Room lifecycle, tourist reconnect token, consented leave (logo + `header-game-leave` confirm in App on Game) |
| [`work-with-game-board`](skills/client/work-with-game-board/SKILL.md) | Board from synced `grid` (`lib/boardGeometry`) + pieceId pieces + presence HUD + focus affordance + shared peek Q&A (topic `peek.md`/`focus.md`) + grille/catapult anim + trap/rescue/push/return + say; brand-logo leave/status in App |
| [`work-with-env-deploy`](skills/client/work-with-env-deploy/SKILL.md) | `VITE_*`, hash router, GitHub Pages |
| [`work-with-test`](skills/client/work-with-test/SKILL.md) | Vitest; ContentUnifiedList/`openRequestType` + AuthorEditTake/`neverLive` + maps SC-MAP-50…52 + AppHeaderChrome (crumbs in `q-page-container` / SC-BRAND-19) + soft-unpublish FollowUp/ModerationUx (SC-PACK-194/195); topic files |

#### Server-wide skills (`.agents/skills/server/`)

Стек: Colyseus 0.18, `@colyseus/tools`, `@colyseus/auth`, `@colyseus/schema`, Express 5, Drizzle + SQLite, mocha + `@colyseus/testing`. Runtime: `../happy-tourist-server/`.

| Skill | Когда |
|-------|--------|
| [`server-align-code`](skills/server/server-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов; handler/lifecycle-петли; first-sync races |
| [`server-locate-change-points`](skills/server/server-locate-change-points/SKILL.md) | Где править (content unified list/favorites/author re-edit/staff take/cancel→draft/set marks without fan-out/`neverLive`/`openRequestType`/D22 cancel restore SC-PACK-196/197; maps statuses; default grants retired SC-PACK-170) |
| [`server-verify-code`](skills/server/server-verify-code/SKILL.md) | Проверка кода на соответствие skills + DRY/KISS/YAGNI |
| [`server-work-with-auth`](skills/server/server-work-with-auth/SKILL.md) | `@colyseus/auth`, Google, mailer, password policy, SPA confirm/reset, `emailVerified`, `htRole`, `BOOTSTRAP_ADMIN_IDS`, `DEFAULT_CONTENT_PACK_IDS` grant **no-op** (SC-PACK-170), JWT `onAuth` |
| [`server-work-with-errors`](skills/server/server-work-with-errors/SKILL.md) | Auth/game failures, HTTP health, client-facing errors |
| [`server-work-with-structure`](skills/server/server-work-with-structure/SKILL.md) | rooms/schema/db/config/lib (content unified list + favorites + author re-edit + staff take + working≠live draft + set marks without fan-out + `neverLive` + cancel restore D22 + `openRequestType`; maps; default grants no-op; support/mailer) |
| [`server-work-with-test`](skills/server/server-work-with-test/SKILL.md) | mocha; zz-contentPacks unify/favorites/author-edit/take/cancel→draft/set marks + fan-out fix/`neverLive` SC-PACK-187…189 + cancel restore SC-PACK-196/197; zz-contentMaps; zz-defaultContentPacks SC-PACK-170; rooms/auth/support |
| [`work-with-rooms`](skills/server/work-with-rooms/SKILL.md) | Room lifecycle (`onDrop`/`onReconnect`/`onLeave` leave-clear holding / `onDispose` clearTurnDeadline + cancel trap pipeline), registration, create via `roomContentSnapshot` (mapId/packId/taskSetIds + chosen maxSeats ≤ map.players) + grille/catapult density / waiting-only seating / phase / seats |
| [`work-with-schema`](skills/server/work-with-schema/SKILL.md) | `@colyseus/schema` sync (`phase` / `maxSeats` / `grid` / `touristsPerPlayer` / peek session / `flippedCells` / `answerCards` + seats pieces by pieceId + turn + `removedTaskKeys` + grilles/catapults) |
| [`work-with-messages`](skills/server/work-with-messages/SKILL.md) | Room `onMessage('move'|'rescue'|'push'|'returnFromFinish'|'peek'|'peekPlace'|'peekSubmit'|'endTurn'|'ready'|'say')` + private `budgets`/`allJailWarning` (pieceId payloads; shared peek; paced trap pipeline; peeks∞) |
| [`work-with-game`](skills/server/work-with-game/SKILL.md) | Waiting-only seating + deferred pieces by pieceId + content snapshot + BoardGeometry + shared peek deck + start/ready/countdown + reconnect + budgets + grille/catapult density + paced trap pipeline + rescue/push/return + `touristMove.ts`; room `tourist` |
| [`work-with-routes`](skills/server/work-with-routes/SKILL.md) | HTTP thin routes; `/api/content/*` unified list (`openRequestType`; working≠live → draft) + favorite + author draft/lock + set `moderationStatus` / `neverLive` ghost + add-task-set cancel restore D22 + staff take/release + maps; collection legacy; grants retired |
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
- Типичный Cursor chat workflow: `/opsx-explore` → [`prepare-changes`](skills/prepare-changes/SKILL.md) (если scope широкий) → `/opsx-propose` → review → `/opsx-apply` → `/opsx-sync` → `/opsx-archive`.
- Артефакты живут в **happy-tourist-meta**, не в siblings. Пути к runtime при apply — через [`docs/projects-map.md`](../docs/projects-map.md) (+ local YAML).
- Always-on: [`AGENTS.md`](../AGENTS.md).
