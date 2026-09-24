---
name: client-locate-change-points
description: Finds files and exact places that need to be changed or where new files should be added in the happy-tourist Vue 3 tourist client based on a task description. Use when the user asks to analyze a task, locate implementation points, find affected files, or identify where changes should be made without editing code.
---

# Locate Change Points

Use this skill when the user gives a task statement and wants only analysis of where changes are needed in this client package (`happy-tourist.github.io`).

Stack: Vue 3 Composition API / `<script setup>`, Vue Router 5 (hash mode), Pinia 4, Quasar 2 + `@quasar/app-vite`, vue-i18n 11, `@colyseus/sdk` 0.18, TypeScript. Skills path for now: `.agents/skills/client/` in this repo (canonical copy may later live under `happy-tourist-meta`).

## Hard Rule

Do NOT edit, create, delete, format, or commit files.

Only inspect the codebase and report:

- existing files that likely need changes;
- exact pages, components, Pinia stores/actions/getters, boot files, routes, i18n, CSS, env/CI, or related server touch points;
- places where new files should be added, if the task requires new code;
- open questions when the implementation location cannot be determined confidently.

## Client Layer Map

Composition is flat under `src/`. Search and assign ownership top-down.

| Layer | Path | Role |
|-------|------|------|
| Routes | `src/router/routes.ts` + `index.ts` | hash routes + guards |
| Pages | `src/pages/*Page.vue` | LoginPage, LobbyPage, Support*, Content*, AdminUsersPage, GamePage |
| Components | `src/components/` | mostly scaffold; prefer pages→stores |
| Boot | `src/boot/` | theme, i18n, colyseus |
| Stores | `src/stores/` | auth, theme, game, support, content, example-store |
| CSS | `src/css/` | app.scss, quasar.variables.scss |
| i18n | `src/i18n/` | en-US |
| Env/CI | `.env.*`, `.github/workflows/deploy.yml` | VITE_*, GitHub Pages |

Allowed dependency direction: `pages` → `stores` / `boot` / `components`. Keep **Colyseus I/O inside Pinia stores** (`auth`, `theme`, `game`, `support`, `content`) — do not scatter `client.*` across many components. Prefer importing `client` from `@/boot/colyseus` over `$colyseus`.

Scaffold leftovers (`EssentialLink.vue`, `example-store.ts`) are not part of the game flow — prefer login / lobby / support / content / game. Dead `pages/index*` were removed with brand cleanup.

### Routes

Router mode: hash (`/#/lobby`, `/#/support`, `/#/content/packs`, `/#/content/staff`, `/#/game/...`). Guards in `src/router/index.ts` await `auth.whenReady()`, then enforce `meta.requiresAuth` / `meta.guest` / `meta.requiresStaff` / `meta.requiresAdmin`.

| Path | Name | Page / meta |
|------|------|-------------|
| `/` → `/lobby` | — | redirect |
| `/login` | `login` | `LoginPage`; `meta.guest` |
| `/lobby` | `lobby` | `LobbyPage`; `meta.requiresAuth` |
| `/support` | `support` | `SupportPage`; `meta.requiresAuth` |
| `/support/staff` | `support-staff` | `SupportStaffPage`; `requiresAuth` + `requiresStaff` |
| `/support/:id` | `support-ticket` | `SupportTicketPage`; `meta.requiresAuth` |
| `/admin/users` | `admin-users` | `AdminUsersPage`; `requiresAuth` + `requiresAdmin` |
| `/content/packs` | `content-catalog` | `ContentCatalogPage`; staff soft-unpublish/republish; `meta.requiresAuth` |
| `/content/collection` | `content-collection` | `ContentCollectionPage`; Edit unpublished creator/staff; soft-unpublished gray non-nav for non-staff; **no** row add-task-set; trash + click isolation (no row `:to`; hide Edit if `blocked`); `meta.requiresAuth` |
| `/content/my-moderation` | `content-my-moderation` | `ContentMyModerationPage`; author «На модерации»; `meta.requiresAuth` |
| `/content/packs/new` | `content-pack-new` | `ContentPackCreatePage`; → working-copy editor; `meta.requiresAuth` |
| `/content/packs/:id` | `content-pack` | `ContentPackPage`; staff Edit+lock + unpublish/republish; add-task-set beside «Задания» (in-catalog); `meta.requiresAuth` |
| `/content/packs/:id/edit` | `content-pack-edit` | `ContentPackEditorPage` (creator working copy / staff save+lock session); `meta.requiresAuth` |
| `/content/packs/:id/add-task-set` | `content-pack-add-task-set` | `ContentPackAddTaskSetPage`; post-publish add-only; `meta.requiresAuth` |
| `/content/packs/:id/tasks/:taskSetId` | `content-pack-tasks` | `ContentPackTasksPage`; nested unpublished/staff session; `meta.requiresAuth` |
| `/content/packs/:id/moderation` | `content-pack-moderation` | `ContentPackModerationPage`; `meta.requiresAuth` |
| `/content/staff` | `content-staff` | `ContentStaffPage`; pending\|needs_revision queue; `requiresAuth` + `requiresStaff` |
| `/content/staff/requests/:id` | `content-staff-request` | `ContentStaffRequestPage`; Approve / needs-revision; no block UI |
| `/content/staff/requests/:id/tasks` | `content-staff-request-tasks` | `ContentStaffTasksPage`; redirect → hub |
| `/game/:roomId` | `game` | `GamePage`; `meta.requiresAuth` |
| `/:catchAll(.*)*` | — | redirect to `/lobby` |

## Decision Guidance

Use these rules to pick the layer before naming files.

### Where does UI live?

| If the change is… | Prefer |
|-------------------|--------|
| A new screen / URL | `src/router/routes.ts` (+ guard meta in `index.ts` if needed) + new `src/pages/FooPage.vue` |
| Page-specific UI / interaction | owning `src/pages/*Page.vue` |
| Reusable across pages | `src/components/` (only when reuse is real; avoid premature extraction) |
| Global shell / brand logo / theme toggle | `src/App.vue` (`q-header` always-button brand logo ≥60px left + Dark toggle + `theme.error` banner; Game leave via logo; auth/other → lobby, lobby noop; syncs `auth` → `theme`) |
| Brand asset / title / favicon | `src/assets/brand/logo.png`; `package.json` `productName` = `Happy Tourist`; `public/favicon.ico` + `index.html` (no PNG favicon set) |
| Theme / Dark preference | `boot/theme.ts`, `stores/theme.ts`, `quasar.config.ts` (`Dark` plugin); guest `localStorage`; registered `GET`/`POST` `/api/theme` (restore ≠ JWT-only) |
| Theme / global styles | `src/css/quasar.variables.scss`, `src/css/app.scss` (`.text-muted`) |
| Copy / locale strings | `src/i18n/` (+ boot `src/boot/i18n.ts` if wiring changes) |

### Pinia as state / Colyseus boundary

Shared mutable session and realtime I/O belong in Pinia, not ad-hoc page-only `client.*` calls.

| Concern | Store surface (typical) |
|---------|-------------------------|
| Auth session | `stores/auth.ts`: `register` / `login` / `loginAnonymously` / `loginWithGoogle` / `logout` / `whenReady`; `isAuthenticated`, `displayName`, `role` / `isStaff` / `isAdmin`; optional `user.theme`; sync via `client.auth.onChange` |
| UI theme (chrome Dark) | `stores/theme.ts`: `syncFromAuthUser` / `toggle`; guest `localStorage` (`ht-theme`); registered `client.http.get('/api/theme')` restore (≠ JWT-only; **no** `auth.user` replace after GET) + `post` on toggle; `App.vue` stable `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` (SC-THEME-10) |
| Support tickets / staff / admin | `stores/support.ts`: HTTP `/api/support/*` + `/api/admin/*` via `client.http` (`change_pack`+`packId`; staff `topic`/`status` query; admin `emailVerified`); pages `Support*` / `AdminUsersPage` |
| Content packs (working copy + unified `submitPack` + add-task-set + staff lock/save + soft-unpublish pack+set/`inCatalog` + cascade yellow + slot chips + add-task-set thread + live drill-in SC-PACK-126…136 + needs_revision / trash/`inCollection` / no block UI) | `stores/content.ts`: HTTP `/api/content/*` via `client.http` (`listMyModeration`, `submitPack`, add-task-set, `acquireEditLock`/`staffSavePack`, `unpublishPack`/`republishPack`, `unpublishTaskSet`/`republishTaskSet`, `needsRevisionRequest`/`cancelRequest`, `loadModeration`/`postModerationMessage`, `deleteUnpublishedPack`/`deleteTaskSet`; `cascadeGapTaskIds`); live `inCollection` + `inCatalog`; map API codes with `contentErrorI18nKey` (incl. `pack_unpublished`, `last_published_task_set`); pages `Content*` (incl. `ContentPackAddTaskSetPage`); create/edit need verified non-anonymous |
| Lobby room list | `stores/game.ts` `subscribeLobby` / `unsubscribeLobby` → LobbyRoom `rooms` / `+` / `-` |
| Enter / leave room | `createGame` / `joinGame` / `leaveGame` (`TOURIST_ROOM` / `LOBBY_ROOM`) |
| Room attach | `_attachRoom` `onStateChange` / `onLeave`; getter `isInRoom` |
| Errors / loading flags | store `error` / `loading` / `listing`; pages show `q-banner`; theme save fail → `App.vue` banner |

Local page state is fine for ephemeral UI (selected cell, form fields) that never leave that page. Game rules and board truth live on the server; client highlights are UI hints only.

### Colyseus / server contract

| If the change is… | Prefer |
|-------------------|--------|
| Client singleton / base URL | `src/boot/colyseus.ts` (`VITE_COLYSEUS_URL`) |
| Auth API usage | `stores/auth.ts` via `client.auth.*` (not `@colyseus/auth` — that package is server-side) |
| Room name, join options, move message, state fields | `stores/game.ts` + matching page; coordinate with sibling `../happy-tourist-server` |
| Env defaults / typed Vite keys | `.env.development`, `.env.production`, `env.d.ts` |
| GitHub Pages build / inject vars | `.github/workflows/deploy.yml` |

Do not invent room schemas, HTTP routes, or move payloads — note the server package when the contract must change.

### Forms, i18n, deploy

- Login / register / guest / Google → `pages/LoginPage.vue` + `stores/auth.ts` + `lib/passwordPolicy.ts` + `components/PasswordStrengthMeter.vue`.
- Forgot / confirm / reset SPA → `pages/ForgotPasswordPage.vue` / `ConfirmEmailPage.vue` / `ResetPasswordPage.vue` + `stores/auth`.
- Cabinet displayName / change-password / email → `pages/AccountPage.vue` + `stores/auth` (`updateDisplayName` / `changePassword` / `canChangePassword`).
- Lobby create / join / list (join busy-lock; clear stale rooms) → `pages/LobbyPage.vue` + `stores/game.ts`.
- Support create / list / thread / staff / admin roles → `pages/Support*.vue` / `AdminUsersPage.vue` + `stores/support.ts` (+ `auth.role` gating; `change_pack` + **in-catalog** pack select).
- Content packs collection (Edit unpublished creator/staff; soft-unpublished gray; **no** row add-task-set; trash/click isolation) / catalog (staff unpublish/republish; «На модерации» hidden for staff) / create / author «На модерации» / live (staff Edit+lock + unpublish/republish; add-task-set beside «Задания» in-catalog only; live slot chips) / working-copy editor (`submitPack` / staff save session; Editor `cascade-gap-outline` CSS) / add-task-set page (slot chips + open-request status/thread/reply) / nested tasks / moderation / staff hub (Approve / needs-revision; slot chips; no block UI) → `pages/Content*.vue` + `stores/content.ts` (+ verify gate / `auth.isStaff`).
- Board interaction / continuous board-busy / presence reserve / `rejoinGame` → `pages/GamePage.vue` + `stores/game.ts`.
- Locale messages → `src/i18n/` (default `en-US`; auth policy/cabinet + support incl. `change_pack` + `content.*` keys incl. `unpublish`/`republish`/`unpublishedByStaff` / `addTaskSet*` / `statuses.needs_revision` / `taskSetStatusMarks.*` / `moderationThread` / `slotEmpty` / `deleteCardConfirm*` by hasLive / trash remove / `errors.*` incl. `pack_unpublished`).
- Unit tests (Vitest) → `vitest.config.ts` + `test/setup.ts` + colocated `src/**/__tests__/*` (incl. `Content*Acl.test.ts`, `ContentSoftUnpublish.test.ts`, `ContentFollowUp4.test.ts`, `ContentFollowUp5.test.ts`, `ContentPackTasksHints.test.ts`, `ContentModerationUx.test.ts` SC-PACK-126…136, `content.simplifyAcl.test.ts`, `SupportChangePack.test.ts`, `support.changePack.test.ts`); store helper `isStaffEditSessionNavigation`; see meta `work-with-test`.
- Deploy / Pages 404 fallback → `.github/workflows/deploy.yml` (`quasar build -m spa`, `index.html` → `404.html`).

## Domain Hotspots

| Domain | Start here |
|--------|------------|
| Auth (email/password policy, anonymous, Google, cabinet profile, logout, role nav) | `pages/LoginPage.vue` / `AccountPage.vue` + `stores/auth.ts` + `lib/passwordPolicy.ts`; router guards in `router/index.ts` |
| Lobby (list / create / join busy-lock; clear stale rooms; quiet resubscribe; Support + «Наборы»→collection) | `pages/LobbyPage.vue` + `stores/game` `subscribeLobby` / `createGame` / `joinGame` |
| Support (tickets / `change_pack` / staff queue / admin roles) | `pages/Support*.vue` / `AdminUsersPage.vue` + `stores/support.ts` |
| Content packs (working copy; add-task-set; staff lock/save; soft-unpublish/`inCatalog`; cascade yellow + SC-PACK-126…136 slot card text / `authorDisplayName` / Tasks `back` / add-task-set thread; needs_revision; trash/`inCollection`; no block UI) | `pages/Content*.vue` + `stores/content.ts` + `ContentFollowUp5.test.ts` |
| Game board (layout, unfinished pieces, continuous board-busy, finish/timeout UX, dual presence rings + seated top-row reserve, strip, turn select/`sendMove`, rejoin) | `pages/GamePage.vue` + `stores/game` `onStateChange` / `sendMove` / `rejoinGame(roomId)` |
| Env / deploy | `.env.*`, `env.d.ts`, `boot/colyseus.ts`, `.github/workflows/deploy.yml` |
| Theme / layout / brand chrome | `App.vue` header (logo + theme + Game status/leave) + `stores/theme` + `boot/theme` + `assets/brand/` + `css/*` (board CSS ≠ app Dark) |
| i18n copy | `src/i18n/`, boot `i18n` |

Today: GamePage board + dual presence rings (outer turn from `turnUntil`/`turnBudgetSeconds`, inner reconnect) + unfinished pieces from synced `seats` (`touristId` + `pieces` (+ `finished`; empty until playing) + `connected` / `reconnectUntil` / `ready` / `finishPlace` / `timeExpired`) / `phase` / `maxSeats` / `countdownRemaining` / `currentTurnSessionId`; top opponents / spectator presence + seated strip (row / HUD ≤~420 → 2×2; no chip/`q-menu`) once own pieces exist; place + timeout + return-confirm modals; countdown overlay; ready affordance; on `isPlaying && isMyTurn && !isMySeatFinished && !isMySeatTimeExpired` select/hints → `sendMove`. Tourist reconnect via `localStorage` token. Room name `tourist`.

## Workflow

1. Read the task statement and extract:
   - target feature or behavior;
   - entities (auth, lobby rooms, game session/board, env/deploy), routes, store actions, UI labels;
   - whether the task changes existing behavior or adds a new flow;
   - whether the Colyseus server contract (room name, state, `move` message, live LobbyRoom listing) is involved.

2. Search by domain terms:
   - route paths and page names (`login`, `lobby`, `game`);
   - store names and actions (`register`, `login`, `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `rejoinGame`, `leaveGame`);
   - Colyseus symbols (`TOURIST_ROOM`, `LOBBY_ROOM`, `client.auth`, `client.reconnect`, `onStateChange`, `room.send`);
   - env keys (`VITE_COLYSEUS_URL`, `VITE_API_URL`);
   - user-visible strings in pages and `src/i18n/`.

3. Walk ownership:
   - route → page → store / boot / component imports;
   - Pinia usage from the page;
   - Colyseus calls only inside `stores/auth`, `stores/theme`, `stores/game`, `stores/support`, or `stores/content` (flag page-level `client.*` as a smell to relocate);
   - server sibling when protocol/schema/room listing must change.

4. Apply decision guidance above to classify each hit as edit vs add, and note related store/boot/env/server touch points.

5. Stop after identifying change points. Do not implement.

## Output Format

Respond in this structure:

```markdown
## Where to edit
- `path/to/file.ts` — why this file is relevant and what kind of change is expected.

## Where to add
- `path/to/NewFile.vue` — why a new file belongs here.

## Related places
- `path/to/related-file.ts` — why it should be checked during implementation.

## Questions
- Clarifying question, only if needed.
```

Omit empty sections. Keep each bullet specific and actionable. Prefer paths under `src/`, plus `.env.*` / `.github/workflows/` when relevant. For server contract work, cite `../happy-tourist-server` under Related (do not edit it from this skill).

## Confidence

Prefer saying "likely" when the location is inferred from naming or nearby patterns. Say "confirmed" only when the inspected code directly proves the relationship.
