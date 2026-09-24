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
| [`client-locate-change-points`](skills/client/client-locate-change-points/SKILL.md) | Где править / куда класть новые файлы (вкл. Google auth, support `change_pack` + content working copy / add-task-set beside «Задания» / staff lock session + soft-unpublish/`inCatalog` + cascade yellow + SC-PACK-126…128 slot chips / add-task-set thread / my-moderation hidden for staff / trash/`inCollection`; Vitest harness incl. `ContentModerationUx`) |
| [`client-verify-code`](skills/client/client-verify-code/SKILL.md) | Проверка кода / compliance skills + DRY/KISS/YAGNI (incl. sticky `.game-hud` presence + sibling avatar / SC-PRESENCE-12) |
| [`colyseus-client`](skills/client/colyseus-client/SKILL.md) | `client.http` + room messages (`move`/`rescue`/`push`/`returnFromFinish`/`peek`/`endTurn`/`say` + private `budgets`/`peekOpen`/`allJailWarning`; `infinite`=peeks∞ only; holes + holding grilles + catapult reveal keys; not axios/BFF) |
| [`client-work-with-auth`](skills/client/client-work-with-auth/SKILL.md) | Colyseus Auth (email/anonymous/Google), password policy + zxcvbn meter, SPA confirm/reset + JSON, cabinet displayName/change-password/`emailVerified`, userdata `role` / `isStaff` / `isAdmin`, once-per-session verify reminder, `onChange`, route guards (`requiresStaff`/`requiresAdmin`); pack create/edit verify modal (lobby/game stay soft) |
| [`client-work-with-errors`](skills/client/client-work-with-errors/SKILL.md) | Store `error` + `q-banner` (pages + App theme; auth/game/support/content), room `onError`; soft-drop rejoin gated by store `consentedLeaving` |
| [`client-work-with-structure`](skills/client/client-work-with-structure/SKILL.md) | pages / components / `src/lib` helpers / boot / stores placement (always-button brand logo ≥60px; Game leave via logo + status in App; auth→lobby; no page «В лобби»; support + content stores; content working copy + add-task-set beside «Задания» + staff Edit lock session + soft-unpublish/`inCatalog` + trash/click isolation / author «На модерации» (hidden for staff) / cascade yellow + SC-PACK-126…128 / author delete unpublished / staff needs_revision / no block UI; password policy helpers) |
| [`work-with-forms`](skills/client/work-with-forms/SKILL.md) | LoginPage / ForgotPasswordPage / ResetPasswordPage / AccountPage / Support create+reply `q-form` (incl. `change_pack` **in-catalog** pack select; content moderation reply Editor/AddTaskSet; policy meter on register/reset/change; `lazy-rules` + clear → `nextTick` → `resetValidation`) / guest + Google buttons |
| [`work-with-pages`](skills/client/work-with-pages/SKILL.md) | Routes + guards login/forgot/confirm/reset/account/lobby/support/content/admin/game; App header (always-button brand logo ≥60px + theme + verify reminder + Game leave via logo/status; auth→lobby; lobby noop; no page «В лобби»; productName/favicon); Lobby create grilleDensity+catapultDensity + join busy-lock + Support + «Наборы»→collection; content working-copy editor + add-task-set beside «Задания» + staff Edit lock session cards↔tasks + unified submit / staff save + soft-unpublish/`inCatalog` + author «На модерации» (hidden for staff) + stable tooltip hints + cascade yellow + Editor/Tasks `cascade-gap-outline` + slot chips + add-task-set thread (SC-PACK-126…128) + trash/click isolation + author delete + staff needs_revision / no block UI; support `change_pack`; staff filters; single ticket Close (`canClose`); admin `emailVerified` badge; GamePage HUD / board |
| [`work-with-stores`](skills/client/work-with-stores/SKILL.md) | Pinia `auth` / `theme` / `game` / `support` (`change_pack`+packId) / `content` (working copy + `submitPack` + add-task-set + staff lock/save + soft-unpublish/`inCatalog` + `isStaffEditSessionNavigation` + cascadeGap* + SC-PACK-126…128 / live `inCollection` / trash confirm / author delete unpublished / no block UI; topic [content.md](skills/client/work-with-stores/content.md)) |
| [`work-with-styles`](skills/client/work-with-styles/SKILL.md) | Quasar Dark + GET/POST `/api/theme`, header always-button brand-logo ≥60px CSS + theme toggle, muted chrome, tourist board (holes + grille overlays / `--grille-anim-ms` 1000) + catapult reveal/broken-hold CSS + top presence + seated strip HUD / budgets beside / end-turn icon / peek-rescue-push top-center / say top↓ own↑ CSS; content `.cascade-gap-outline` (SC-PACK-126) |
| [`work-with-localization`](skills/client/work-with-localization/SKILL.md) | vue-i18n boot; auth-email RU; password policy keys; `support.*` incl. topic `change_pack`; `content.*` («Набор карточек» / `addTaskSet*` / `unpublish`/`republish`/`unpublishedByStaff` / `staffEditSubtitle` / `tasksSaveHint` / `questionNeedsSlot` / `taskSetStatusMarks.*` / `moderationThread` / `slotEmpty` / `deleteCardConfirm*` by hasLive / `myModeration*` / `statuses.needs_revision` / `content.errors.*` incl. `pack_unpublished`); game keys |
| [`work-with-lobby`](skills/client/work-with-lobby/SKILL.md) | Live LobbyRoom subscribe, create-with-maxSeats + grilleDensity + catapultDensity modal (12/22/35% seed each; no Play shortcut), join busy-lock, clear stale rooms until snapshot, quiet resubscribe; header links Support + «Наборы» → `content-collection` |
| [`work-with-rooms`](skills/client/work-with-rooms/SKILL.md) | Room lifecycle, tourist reconnect token, consented leave (brand-logo confirm in App on Game) |
| [`work-with-game-board`](skills/client/work-with-game-board/SKILL.md) | Board + top opponents / spectator presence + seated always reserves top row height + `.game-hud` strip (row/2×2; no chip/`q-menu`) + grille (`GRILLE_ANIM_MS=1000`; defer drop during catapult hops) + catapult land→overlay→fling (`CATAPULT_ANIM_MS=1000`, broken 300+300, spectator parity, D13 atomic mirror, **finish travel after vanish** even if finish sync late — SC-FINISH-20 / SC-BOARD-31, continuous board-busy intent→idle) + trap/rescue/push + return strip icon (no modal) + dual rings + budgets beside + end-turn icon right-center + peek/rescue/push top-center + holes + say top↓/own↑ + nearest-center finish + move/push finish travel (`lastKnown` seed, not `finishAnimFrom` at submit) + return anim; brand-logo leave/status in App |
| [`work-with-env-deploy`](skills/client/work-with-env-deploy/SKILL.md) | `VITE_*`, hash router, GitHub Pages |
| [`work-with-test`](skills/client/work-with-test/SKILL.md) | Vitest + `@vue/test-utils` (harness live; stores/pages `auth`/`theme`/`game`/`support`/`content` simplify ACL SC-PACK-100…128 + `ContentSoftUnpublish` + `ContentModerationUx` SC-PACK-126…128 + SupportChangePack SC-SUP-27…29; topic files) |

#### Server-wide skills (`.agents/skills/server/`)

Стек: Colyseus 0.18, `@colyseus/tools`, `@colyseus/auth`, `@colyseus/schema`, Express 5, Drizzle + SQLite, mocha + `@colyseus/testing`. Runtime: `../happy-tourist-server/`.

| Skill | Когда |
|-------|--------|
| [`server-align-code`](skills/server/server-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов; handler/lifecycle-петли; first-sync races |
| [`server-locate-change-points`](skills/server/server-locate-change-points/SKILL.md) | Где править / куда класть новые файлы (вкл. `config/auth`, `lib/passwordPolicy`, `lib/support` `change_pack` in-catalog, `lib/content` working copy + submitPack/add-task-set + staff lock/save + soft-unpublish/`in_catalog` + needs-revision + cascadeNormalize + author delete/my-moderation, support/content/admin + auth profile HTTP) |
| [`server-verify-code`](skills/server/server-verify-code/SKILL.md) | Проверка кода на соответствие skills + DRY/KISS/YAGNI |
| [`server-work-with-auth`](skills/server/server-work-with-auth/SKILL.md) | `@colyseus/auth`, Google OAuth, smtp.bz mailer, password policy, SPA mail links + JSON confirm/reset, displayName persist + change-password/bumpTokenVersion, `emailVerified` (soft for rooms; hard for content pack create/edit/submit), `htRole`→userdata `role`, `BOOTSTRAP_ADMIN_IDS`, send-confirm / change-email, JWT `onAuth` |
| [`server-work-with-errors`](skills/server/server-work-with-errors/SKILL.md) | Auth/game failures, HTTP health, client-facing errors |
| [`server-work-with-structure`](skills/server/server-work-with-structure/SKILL.md) | rooms / schema / db / config / lib (mailer + support `change_pack` in-catalog + content working copy + submitPack/add-task-set + staff lock/save + soft-unpublish/`in_catalog` + needs-revision + cascadeNormalize ≠ false `answers_dirty` + passwordPolicy) / app.config placement |
| [`server-work-with-test`](skills/server/server-work-with-test/SKILL.md) | mocha + `@colyseus/testing` (SC-PIECE + SC-MOVE + SC-BOARD + SC-FINISH + SC-SAY + lobby density + theme + auth + `test/support.test.ts` SC-SUP incl. change_pack + soft-unpublished reject + `zz-contentPacks.test.ts` SC-PACK-100…114 + soft-unpublish 120…124 working copy/submit/add-task-set/staff lock) |
| [`work-with-rooms`](skills/server/work-with-rooms/SKILL.md) | Room lifecycle (`onDrop`/`onReconnect`/`onLeave` leave-clear holding / `onDispose` clearTurnDeadline + cancel trap pipeline), registration, maxSeats / grilleDensity / catapultDensity / waiting-only seating / phase / seats |
| [`work-with-schema`](skills/server/work-with-schema/SKILL.md) | `@colyseus/schema` sync (`phase` / `maxSeats` / countdown + `seats` + connectivity/`ready`/`finishPlace`/`timeExpired` + piece `finished`/`trapped` + turn fields + `removedTaskKeys` + `holdingGrilleKeys` + `revealingCatapultKeys` / `brokenCatapultKeys`) |
| [`work-with-messages`](skills/server/work-with-messages/SKILL.md) | Room `onMessage('move'|'rescue'|'push'|'returnFromFinish'|'peek'|'peekAnswer'|'endTurn'|'ready'|'say')` + private `budgets`/`peekOpen`/`allJailWarning` (trap/rescue/push/return + paced trap pipeline/catapult + idle re-eval SC-MOVE-93; peeks∞; holes reject land) |
| [`work-with-game`](skills/server/work-with-game/SKILL.md) | Waiting-only seating + deferred pieces + start/ready/countdown + reconnect + private steps/peeks∞ + grille+catapult density 12/22/35%/paced land-resolve + deferred turn + idle re-eval (SC-MOVE-93)/fling/trap/rescue/push/return/all-jail/leave-clear + become-current grant + multi peek + step-loss + holes not landable + turn deadlines + center finish + `touristMove.ts`; room `tourist` |
| [`work-with-routes`](skills/server/work-with-routes/SKILL.md) | HTTP `createEndpoint` / thin routes (incl. theme, auth email JSON, `/api/support/*` create-ack + `change_pack`/`packId`, `/api/content/*` working-copy draft + unified submit + add-task-set + edit-lock/staff-save + soft-unpublish/republish + needs-revision + my-moderation + staff pending queue; block endpoints retained, `/api/admin/*`) |
| [`work-with-middleware`](skills/server/work-with-middleware/SKILL.md) | CORS, `/health`, monitor/playground |
| [`work-with-config`](skills/server/work-with-config/SKILL.md) | env, secrets, OAuth + email `config/auth`, `BOOTSTRAP_ADMIN_IDS`, `defineServer` |
| [`work-with-database`](skills/server/work-with-database/SKILL.md) | GameDatabase, users schema (incl. nullable `theme`, `emailVerified`, `htRole`) + support_tickets/messages (`pack_id` for `change_pack`) + content_* pack tables (`working_revision_id`, `edit_locked_*`, `in_catalog` soft-hide, moderation `type` pack\|task_set, open pending\|needs_revision; migrate drop `content_user_drafts`; cascade slot clear ≠ `answers_dirty`; author delete; `ensureContentTables`) |
| [`work-with-env-deploy`](skills/server/work-with-env-deploy/SKILL.md) | VPS, PM2, GitHub Actions rsync; smtp.bz + `AUTH_BACKEND_URL` / `CLIENT_APP_URL` / `BOOTSTRAP_ADMIN_IDS` |
| [`work-with-loadtest`](skills/server/work-with-loadtest/SKILL.md) | `@colyseus/loadtest` scripts |

Runtime-пути в skills (`src/…`) — относительно корня соответствующего sibling-репозитория.

Kilo Code: при появлении канона [`../kilo.template.jsonc`](../kilo.template.jsonc); локально `cp kilo.template.jsonc kilo.jsonc` (ignored). Nested skills — `skills.paths` в рабочем `kilo.jsonc`.

---

## OpenSpec / workflow

- Конфиг: [`openspec/config.yaml`](../openspec/config.yaml) (`projectsMap`, `context`, capability rules).
- Capability ID = продуктовый путь (auth/login, lobby/rooms, game/move, content/packs, …), не имя change.
- Delta: `openspec/changes/<slug>/specs/<capability-id>/spec.md`
- Main: `openspec/specs/<capability-id>/spec.md`
- Типичный Cursor chat workflow: `/opsx-explore` → `/opsx-propose` → review → `/opsx-apply` → `/opsx-sync` → `/opsx-archive`.
- Артефакты живут в **happy-tourist-meta**, не в siblings. Пути к runtime при apply — через [`docs/projects-map.md`](../docs/projects-map.md) (+ local YAML).
- Always-on: [`AGENTS.md`](../AGENTS.md).
