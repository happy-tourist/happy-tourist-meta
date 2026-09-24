---
name: client-work-with-structure
description: >-
  Use when placing or moving UI in the happy-tourist Vue 3 client: pages
  (*Page.vue), components, Pinia stores, Quasar boot files, dependency
  direction between layers, or deciding whether new UI belongs in pages vs
  components vs stores vs App shell (always-button brand logo ≥60px; Game leave via logo + status;
  auth→lobby; theme; no page «В лобби»; content packs working copy + add-task-set beside «Задания» + staff Edit lock session + soft-unpublish pack+set/`inCatalog` + confirm on unpublish + live summary/drill-in + AddTaskSet chips (SC-PACK-126…133) + trash/click isolation (no row `:to`; soft-unpublished gray; hide Edit if `blocked`) / author «На модерации» (hidden for staff) / cascade yellow + Editor/Tasks `cascade-gap-outline` CSS / slot chips on question lists / add-task-set status+thread / author delete unpublished / staff needs_revision / no block UI). No blocks/
  or dialogs/ registry layers.
---

# Work With Structure

## Overview

Use this skill when creating or relocating UI under `src`. Feature UI is
layered as **pages → stores / boot / components**. There are **no** `blocks/`
or `dialogs/` registry layers (unlike B2B).

Stack: Vue 3 Composition API (`<script setup>`), TypeScript, Vue Router 5
(hash), Pinia 4, Quasar 2 (`@quasar/app-vite`), vue-i18n, `@colyseus/sdk`.

Path alias: `@/` → `src/`.

Skills for this client live under `.agents/skills/client/` (this package; meta
may also host copies later).

## Core Rules

1. Before adding folders, inspect a nearby peer in the **same layer** and match its file name and import style.
2. Create only the files the feature needs. Do not add empty stubs “for later”.
3. Place UI by layer role (route page vs reusable widget vs store-owned I/O).
4. Respect **allowed dependency direction** (see below). Never invert layers.
5. Import `.vue` / `.ts` by direct path. No per-feature `index.ts` barrels required.
6. Keep `App.vue` as the shared shell: `q-layout` → shared `q-header` (brand logo ≥60px left always as the **same** interactive button; theme toggle always; on **Game** also centered match status; leave = logo click + confirm dialog) → `q-page-container` → theme `q-banner` + `<router-view />`. Logo click modes: auth → lobby / lobby noop / Game leave / other authenticated → lobby (never swap to bare decorative `img`). Lobby logout and page banners stay **inside each page**; do **not** reintroduce Material `logout` leave, page-level «В лобби», or a page-local Game leave/status header; do not duplicate the theme toggle per page.
7. Keep Colyseus auth/room/theme/support/content I/O inside Pinia stores (`auth`, `theme`, `game`, `support`, `content`). Prefer `import { client } from '@/boot/colyseus'` over `$colyseus` in script.
8. Use Quasar auto-imported components (`q-page`, `q-btn`, …). Do not manually register Quasar UI components.
9. Prefer Composition API + `<script setup lang="ts">`. Do not introduce Options API pages.

## Folder Map (`src`)

| Layer | Path | Role |
|-------|------|------|
| Pages | `src/pages/*Page.vue` | Route-level screens; compose stores + Quasar + optional components |
| Components | `src/components/` | Reusable widgets (e.g. `PasswordStrengthMeter.vue`; plus Quasar scaffold leftovers) |
| Lib | `src/lib/` | Pure helpers without Pinia/Colyseus I/O (`passwordPolicy.ts`, `passwordStrength.ts`) |
| Stores | `src/stores/` | Pinia: `auth`, `theme`, `game`, `support`, `content` (+ unused scaffold `example-store`) |
| Boot | `src/boot/` | Quasar boot: `theme`, `i18n`, `colyseus` (registered in `quasar.config.ts`; `framework.plugins: ['Dark']`) |
| Router | `src/router/` | `routes.ts` + guards in `index.ts` (`filenameBasedRouting: false`) |
| i18n | `src/i18n/` | Locale message trees (`en-US`) |
| CSS | `src/css/` | `app.scss`, `quasar.variables.scss` |
| Assets | `src/assets/` | Static assets (`brand/logo.png`, `tourists/…`, `grilles/grille.png`, `catapults/catapult.png` + `catapult-broken.png`, …) |

Outside `src`: `public/` (product `favicon.ico` only — no scaffold PNG icon set), `quasar.config.ts`, `package.json` (`productName` = `Happy Tourist`), `.env.development` / `.env.production`, `.github/workflows/`.

**Not used in this project:** `src/blocks/`, `src/dialogs/`, Vuex, axios BFF layer, `*View` page suffix.

Scaffold leftovers (`EssentialLink.vue`, `example-store.ts`) — prefer the login / forgot / account / lobby / game flow; do not extend scaffold paths for new features. Dead `pages/index*` and Quasar logo SVG were removed.

## Dependency Direction

Allowed:

```text
pages       →  stores / boot / components / router (params)
components  →  other components (keep lean; prefer props over store)
stores      →  boot/colyseus (client); theme store also uses boot/theme helpers
boot        →  env / SDK / i18n / early Dark apply only
App.vue     →  layout + shared header (always-button brand logo ≥60px; theme; Game status; leave via logo; auth→lobby / lobby noop) + banner + auth→theme sync + game leave
```

Also normal:

- Pages call Pinia actions (`useAuthStore`, `useGameStore`) and read store state.
- `App.vue` uses `useThemeStore` / `useAuthStore` for the shared theme toggle + brand logo nav, and on Game also `useGameStore` for leave (logo) + match status (not pages).
- Router guards await `useAuthStore().whenReady()` then enforce `requiresAuth` / `guest`.
- Quasar components used in templates without local imports (auto-import).

**Forbidden inversions:**

| Wrong | Why |
|-------|-----|
| `components` → `pages` | Widgets stay page-agnostic |
| `stores` → `pages` / `components` | Data layer must not import UI |
| `boot` → `pages` / `components` | Boot is app setup only |
| Scattering `client.*` across many components | Keep Colyseus I/O in `stores/auth`, `stores/theme`, `stores/game`, `stores/support` |
| Adding `blocks/` or `dialogs/` registry “like B2B” | This app has no those layers |

## Where New UI Belongs

Decide in this order:

1. **New route / screen?** → `src/pages/<Name>Page.vue` + route in `src/router/routes.ts`
2. **Reusable control used in 2+ pages or clearly generic?** → `src/components/<Name>.vue` (or small folder if peers do)
3. **Auth / room / move / listing / theme preference logic?** → Pinia store (`stores/auth.ts`, `stores/theme.ts`, or `stores/game.ts`), not inline in the page beyond thin wiring
4. **App-wide plugin / SDK singleton / early Dark apply?** → Quasar boot file under `src/boot/` + register in `quasar.config.ts`
5. **Page-local overlay / dialog?** → Keep in the page (or extract a component) with local `ref` / Quasar dialog props — **do not** invent a global dialogs registry. Exception: Game leave confirm lives in `App.vue` (shared shell), not `GamePage`.

### Pages vs components — what belongs where

**Put in pages**

- Route entry (`*Page.vue`) and page orchestration (form state, selection, route params).
- Screen chrome for that route (title bar, Lobby logout). Brand logo + Game leave/status live in `App.vue`, not on `GamePage`; do not add page «В лобби».
- Markup that exists only on that route (lobby list, board + sticky `.game-hud`, login forms).
- Thin wiring: call store actions, show page `store.error` via `q-banner`.

**Put in components**

- Small reusable widgets shared by 2+ pages (or clearly extractable board/cell chrome once reused).
- Prefer extracting only when reuse or page size justifies it — pages may own substantial markup today.

**Put in stores**

- Colyseus auth: register / login / anonymous / Google / logout / `whenReady`.
- Theme preference: Quasar Dark + guest `localStorage` / registered `GET`+`POST` `/api/theme` (restore ≠ JWT-only; `stores/theme.ts`).
- Room lifecycle: `subscribeLobby`, `unsubscribeLobby`, `createGame`, `joinGame`, `leaveGame`.
- Mirrored room state: `board`, `myColor`, `currentTurn`, `status`, `rooms`, errors.

**Put in boot**

- `theme` — early `Dark.set` from `localStorage` (`ht-theme`) or `auto`.
- `i18n` — `createI18n` + `app.use`.
- `colyseus` — `new Client(...)`, export `client`, optional `$colyseus` globalProperty.

### What NOT to put

| Avoid | Prefer |
|-------|--------|
| Per-page theme toggle or theme HTTP | Shared `App.vue` header + `stores/theme` |
| Colyseus `client.create` / `send` / `auth.*` / theme POST in a random component | `stores/auth`, `stores/theme`, or `stores/game` |
| New `*View.vue` naming | `*Page.vue` |
| `src/blocks/` or `src/dialogs/index` registry | Page-local UI or a plain component |
| Empty layer folders “for later” | Add when the first file is needed |
| Extending unused scaffold pages / Quasar logo / PNG favicon set | Add `*Page.vue` + `routes.ts`; brand logo in App; `productName` + `favicon.ico` only |
| Manual Quasar component registration | Auto-import |
| New axios/API module for Colyseus HTTP | LobbyRoom via store (`subscribeLobby`); `client.http` only as unused fallback |

## Naming

| Kind | Convention | Examples |
|------|------------|----------|
| Page file | PascalCase + `Page` suffix | `LoginPage.vue`, `ForgotPasswordPage.vue`, `ConfirmEmailPage.vue`, `ResetPasswordPage.vue`, `AccountPage.vue`, `LobbyPage.vue`, `GamePage.vue` |
| Route `name` | lowercase (existing) | `login`, `lobby`, `game` |
| Component file | PascalCase | `EssentialLink.vue` (scaffold); new: `BoardCell.vue` |
| Store file | kebab or descriptive | `auth.ts`, `theme.ts`, `game.ts` |
| Boot file | lowercase | `theme.ts`, `i18n.ts`, `colyseus.ts` |
| Imports | Alias `@/` | `import { client } from '@/boot/colyseus'` |

Pages are **flat files** under `src/pages/` (not `pages/LoginPage/LoginPage.vue` folders), matching current peers.

## Typical Shapes

### Page

```text
src/pages/LobbyPage.vue   # <script setup lang="ts"> + Quasar template
```

Existing pages: `LoginPage`, `ForgotPasswordPage`, `ConfirmEmailPage`, `ResetPasswordPage`, `AccountPage`, `LobbyPage`, `GamePage`.

### Component (when extracted)

```text
src/components/BoardCell.vue
```

### Store

```text
src/stores/auth.ts    # setup store
src/stores/theme.ts   # setup store — Quasar Dark preference
src/stores/game.ts    # options store
src/stores/index.ts   # Quasar Pinia entry
```

### Boot

```text
src/boot/theme.ts
src/boot/i18n.ts
src/boot/colyseus.ts
```

Registered in `quasar.config.ts` boot array: `theme`, `i18n`, `colyseus` (+ `framework.plugins: ['Dark']`).

## Real Composition Examples

**Auth pages** — `LoginPage` / `ForgotPasswordPage` / `ConfirmEmailPage` / `ResetPasswordPage` / `AccountPage` use `useAuthStore()` (register / login / anonymous / Google / forgot / confirmEmail / resetPassword / send-confirm / change-email); show `error` banner / dialogs; no Colyseus calls outside the store. Confirm/reset product UX is **SPA + JSON** (`/#/confirm-email`, `/#/reset-password`) — not API HTML.

**Lobby** — `LobbyPage` uses `useGameStore().subscribeLobby` / `createGame({ maxSeats, grilleDensity, catapultDensity })` / `joinGame` and `useAuthStore` for display/logout.

**Game** — `GamePage` shows tourist board (scroll region) + top presence (opponents / spectator all) + sticky seated `.game-hud` (own + strip row/2×2; dual rings; **no** chip/`q-menu`) + grille overlays from `holdingGrilleKeys` (`GRILLE_ANIM_MS=1000`; defer drop while catapult queue busy) + catapult sequential overlays from `revealingCatapultKeys` / `brokenCatapultKeys` (`CATAPULT_ANIM_MS=1000`, land→overlay→fling, D13 atomic mirror, board-busy incl. pending grille) + push icons + return strip icon (no confirm modal) when seated; on `isMyTurn` local select/hints and `game.sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish`; seated+online own marker may `game.sendSay` (preset bubbles from `sayEvents`); `rejoinGame(roomId)` on mount / soft-fail gated by store `consentedLeaving` (reconnect token → `joinById`).

**App shell** — `App.vue` hosts `q-layout` → shared `q-header` (always-button brand logo ≥60px left; theme toggle; on Game centered status; leave via logo click; auth/other → lobby; lobby noop) → leave confirm dialog → `router-view`; stable `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `theme.syncFromAuthUser` (GET restore for registered; do not replace `auth.user` after GET); shows `theme.error` banner; Game leave → `leaveGame` → lobby.

**Boot** — `theme.ts` applies early Dark (`readStoredTheme` / `clearStoredTheme`); `colyseus.ts` exports singleton `client`; stores import them.

## Creating New Pieces — Checklist

**New page**

1. `src/pages/<Name>Page.vue` with `<script setup lang="ts">`.
2. Add route in `src/router/routes.ts` (lazy `() => import('@/pages/...')`), set `meta.requiresAuth` or `meta.guest` as needed.
3. Wire UI to Pinia; keep Colyseus I/O in stores.
4. Keep Lobby/Login route chrome in the page; shared brand logo + theme toggle + Game leave/status stay in `App.vue` (no page «В лобби»).

**New component**

1. `src/components/<Name>.vue`.
2. Import from pages (or other components); do not import pages/stores unless clearly justified — prefer props/emits.
3. Do not call Colyseus directly; emit events or accept callbacks from the page/store consumer.

**New store logic**

1. Extend `stores/auth.ts`, `stores/theme.ts`, `stores/game.ts`, `stores/support.ts`, or `stores/content.ts` (or add a focused store if domain is new).
2. Import `client` from `@/boot/colyseus` when calling the SDK.
3. Surface `error` / loading flags for pages (or `App.vue` for theme) to display.

**New boot file**

1. `src/boot/<name>.ts`.
2. Register in `quasar.config.ts` boot list.
3. Export symbols for stores/pages to import; avoid putting feature UI there.

## Domain Anchors

| Domain | Page | Store / boot |
|--------|------|----------------|
| Auth | `LoginPage` / `ForgotPasswordPage` / `ConfirmEmailPage` / `ResetPasswordPage` / `AccountPage` | `stores/auth` + `boot/colyseus`; policy/meter: `lib/passwordPolicy` + `components/PasswordStrengthMeter` |
| Theme (chrome Dark) | `App.vue` header | `stores/theme` + `boot/theme` |
| Lobby / rooms | `LobbyPage` | `stores/game.subscribeLobby` / create `{ maxSeats, grilleDensity, catapultDensity }` / join |
| Game session | `GamePage` | `stores/game` room attach + seats / grilles / catapults / top + seated-HUD presence / say / strip + rescue/push + return strip icon (no modal); soft-drop gated by `consentedLeaving` |
| Game leave + match status | `App.vue` header (logo leave on Game; status on Game) | `stores/game` `leaveGame` + status / phase getters |
| Content packs | `ContentCatalogPage` (staff unpublish/republish + confirm) / `ContentCollectionPage` (Edit unpublished creator/staff; staff unpublish/republish + confirm; soft-unpublished gray; **no** row add-task-set; trash; no row `:to`) / `ContentMyModerationPage` / `ContentPackPage` (staff Edit+lock; pack+set unpublish/republish; set summary + drill-in; add-task-set beside «Задания») / `ContentPackCreatePage` / `ContentPackEditorPage` (set soft-hide; `cascade-gap-outline` CSS on task-set rows) / `ContentPackAddTaskSetPage` (slot chips + open-request status/thread/reply + rounded answer chips) / `ContentPackTasksPage` (live drill-in; task yellow + slots; set unpublish inside Edit) / `ContentPackModerationPage` / `ContentStaffPage` / `ContentStaffRequestPage` (Approve / needs-revision; **slot chips**; **no** unpublish; no block UI) / `ContentStaffTasksPage` (→ hub) | `stores/content` HTTP (`client.http`); working copy + `submitPack` + add-task-set + staff lock session + soft-unpublish pack+set (`unpublishPack`/`republishPack`/`unpublishTaskSet`/`republishTaskSet`, `inCatalog`) + cascadeGap* + SC-PACK-126…133 + needs_revision |
| Support | `SupportPage` / `SupportTicketPage` / `SupportStaffPage` | `stores/support` |
| Brand / title / favicon | `App.vue` + `package.json` / `index.html` / `public/favicon.ico` | `assets/brand/logo.png`; `productName` Happy Tourist; single favicon |
| Shell | `App.vue` | layout + always-button brand logo ≥60px + theme header/banner + Game leave/status + `router-view` |

Routes (from `src/router/routes.ts`):

| Path | Name | Meta |
|------|------|------|
| `/` → `/lobby` | — | redirect |
| `/login` | `login` | `guest` |
| `/lobby` | `lobby` | `requiresAuth` |
| `/game/:roomId` | `game` | `requiresAuth` |
| `/:catchAll(.*)*` | — | → `/lobby` |

Router mode: **hash** (`/#/lobby`, `/#/game/...`).

Synced board encoding stays client-local (`LAYOUT`); authority for seating/turn/move is server schema + `move` message.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Naming a screen `*View.vue` | Use `*Page.vue` |
| Adding `blocks/` or a dialogs registry | Keep UI in page/component |
| Putting room/auth/theme/support/content SDK calls in the page body | Move to `stores/auth` / `stores/theme` / `stores/game` / `stores/support` / `stores/content` |
| Collection list row with `:to` plus side Edit/trash | Quasar router-link races icons — use `@click` navigate + `@click.stop` on side actions (`work-with-pages`) |
| Showing Edit on a `blocked` pack (collection or live) | Hide Edit when `blocked` (SC-PACK-25); collect may stay disabled |
| Importing a page from a component | Invert: page imports the component |
| Duplicating theme toggle on every page | Keep shared toggle in `App.vue` |
| Reintroducing page-local Game leave/status or Material `logout` leave | Keep leave via brand logo + status in `App.vue` on Game (`work-with-pages`) |
| Swapping logo button ↔ bare `img` / decorative auth | Keep one always-button control (≥60px); auth/other → lobby, lobby noop (`work-with-pages`) |
| Adding page «В лобби» buttons | Use App brand logo (`auth.backToLobby` aria-only) |
| Using removed scaffold `pages/index*` / Quasar logo for new routes | Add `*Page.vue` + `routes.ts` entry |
| Reintroducing Vuex or axios for Colyseus | Pinia + `client` / `client.http` |
| Manual Quasar imports for auto-imported tags | Use `q-*` in template as-is |

## Related Skills

- Page routing / guards → `work-with-pages`
- Pinia auth/theme/game → `work-with-stores`
- Quasar Dark / muted chrome → `work-with-styles`
- Sibling server contracts → `../happy-tourist-server` (coordinate room name `tourist`, move payload, state shape, `/api/theme`)

## Verification

For structure-only placement tasks, confirm:

- [ ] Correct layer (`pages` / `components` / `stores` / `boot`)
- [ ] `*Page.vue` naming (not `*View`)
- [ ] No new `blocks/` or `dialogs/` registry
- [ ] Dependency direction respected
- [ ] Colyseus I/O stays in stores (`auth` / `theme` / `game` / `support` / `content`)
- [ ] Route registered in `routes.ts` with correct meta
- [ ] Shared always-button brand logo ≥60px + theme chrome stay in `App.vue`; Game leave via logo + status stay in App; auth/other → lobby / lobby noop; no page «В лобби»; other route chrome stays in pages
