## Context

Client-only UX polish on catalog lists after `guest-auto-entry-catalog-ux`: guests still see «Все», non-staff UI may still show СНЯТО if the client does not omit soft-unpublished rows (server list already staff-only), and add-task-set is hidden for anonymous instead of using the existing create gate. Light/dark Maps mocks + prepare-mock extract define measurable page chrome; pack list mirrors the same chrome.

**Пакет:** client (`../happy-tourist.github.io`). Server create/add-task-set reject for anonymous — без изменений.

## Goals / Non-Goals

**Goals:**

- Hide all filter chrome for `auth.user?.anonymous` on packs + maps lists.
- Hide soft-unpublished (`hasLive && inCatalog === false`) cards from non-staff on those lists; staff keep СНЯТО chrome.
- Show «Добавить набор заданий» to anonymous and unverified on live pack → existing login/verify gate; keep view-only task-set entry (live boot skips auth gates).
- Align packs/maps list title band + create with Visual Spec (below); keep logo without wordmark; keep map capacity rows.

**Non-Goals:**

- Wordmark in header; map-card «2-4» pill redesign; soft-unpublished task-set cards on pack detail; server list API shape / soft-unpublish rules (already staff-only on list); prepare-mock/verify-mock as separate apply steps beyond implementing Visual Spec.

## Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| D1 | Guest = `anonymous` only for filter hide | Explore: «для всех гостей»; unverified keeps identity filters |
| D2 | Soft-unpublished omit on list: client defense-in-depth | Server `listCatalog` / `listMaps` already return soft-unpublished **only to staff** (authors get own never-published, not soft-unpublished). Client still filters `hasLive && inCatalog === false` for `!isStaff` so UI stays correct if payload ever includes them; **no** list-API shape change |
| D3 | Reuse pack/map `gateOpen` / `gateMode` for add-task-set | Anonymous **and** unverified (`needsEmailVerification`) **MUST see** CTA (parity with list create). Click → existing `ensureEligible` login/verify gate; MUST NOT open editor. No new modal copy family. `showAddTaskSet` drops only the anonymous/unverified exclusions (still hide for staff / blocked / not in-catalog) |
| D4 | No header wordmark | Preserve SC-BRAND-01; mock wordmark out of scope |
| D5 | Keep map capacity rows | SC-MAP-55; mock «2-4» out of scope |
| D6 | Pack detail soft-unpublished task sets unchanged | Scope = catalog lists only |
| D7 | Title/create layout: keep title+create row; hide filter row for guest | Matches mock composition when «Все» gone; empty `packs-filters` / `maps-filters` wrapper MUST NOT remain for guest (`v-if="!isGuest"` on the row) |
| D8 | Live view-only bypasses auth gates | Today `ContentPackTasksPage.load()` calls `checkGate()` first — blocks **anonymous** (login) and **unverified** (verify) before any load. For `query.view=live` (pack already navigates with `{ view: 'live' }`) on an in-catalog published set, MUST enter `liveViewMode` **without** opening login/verify dialogs. Call `checkGate` only for mutate paths (compose/save/delete). Soft-unpublished non-enterable for non-staff unchanged |

### Точки врезки (client)

| Area | Files (indicative) |
|------|-------------------|
| Packs list filters + soft-unpub filter | `src/pages/ContentCatalogPage.vue` (`isGuest`, whole filter row `v-if`, `filteredPacks` omit `hasLive && inCatalog === false` for `!isStaff`) |
| Maps list filters + soft-unpub filter | `src/pages/MapsListPage.vue` (same pattern) |
| Add-task-set CTA + gate | `src/pages/ContentPackPage.vue` (`showAddTaskSet` include anonymous + unverified; `onAddTaskSet` → existing `ensureEligible` / `gateOpen`) |
| Task-set view-only | `src/pages/ContentPackTasksPage.vue` — D8: skip `checkGate` on `view=live` boot; keep `liveViewMode` + hide compose; mutate still gated |
| Page chrome / tokens | `ContentCatalogPage.vue`, `MapsListPage.vue`, `src/css/app.scss` / Quasar vars as needed; breadcrumbs remain `App.vue` |
| Tests | `ContentUnifiedList.test.ts`, `ContentMaps.test.ts`, `ContentPackAcl.test.ts` / pack detail guest tests + guest live task-set; `AppHeaderChrome` if title chrome assertions land there |
| Meta skills (после UI) | `work-with-pages/content-pages.md`, `work-with-stores/content.md`, `work-with-stores/maps.md`, `client-work-with-test/stores.md` — сейчас канон «**all** stays» (SC-PACK-286/SC-MAP-69); sync в `tasks.md` §6 |

Слои: pages → auth store flags; no new Pinia API required. Soft-unpublished list predicate: `hasLive && inCatalog === false` (strict `=== false`, не `!inCatalog`). Checklist реализации — `tasks.md`.

## Server contract (unchanged — reuse as-is)

Client-only change; **no** new room messages, HTTP routes, schema fields, DB migrations, or auth API. Catalog guest chrome / СНЯТО omit / add-task-set gate / view-only task-set reuse existing server surfaces:

| Surface | Role for this change |
|---------|----------------------|
| `GET /api/content/packs` (`listCatalog`) + `GET /api/content/maps` (`listMaps`) with `auth.middleware()` | JWT required (incl. anonymous). Soft-unpublished (`hasLive` / `liveRevisionId` + not in catalog) returned **only to staff**; authors get own never-published, not soft-unpublished. Client D2 omit is defense-in-depth — **no** list-API shape or filter query params |
| `GET /api/content/packs/:id` (`getLivePack`) | Anonymous JWT MAY load in-catalog non-blocked live (incl. published task sets for client view-only). Soft-unpublished → `pack_unpublished` for non-staff (unchanged) |
| Pack/map create + add-task-set (`requireVerifiedEditor` → `registered_user_required` when `anonymous === true`; `email_verified_required` when unverified) | Guest create / add-task-set / submit stay denied — do **not** widen. Client login/verify gate is UX only; server reject remains authority for SC-PACK-165 |
| Colyseus rooms / schema / messages | Out of scope — no touch |

Out of scope remains: server-side filter query params, soft-unpublish rule changes, OAuth providers, staff moderation redesign, mocha expectations beyond “leave existing reject tests green”.

## Migration Plan

Client-only deploy. No DB migration. Rollback = revert client. Existing anonymous JWT + list/detail gates unchanged.

## Risks / Trade-offs

- If a non-staff client still shows СНЯТО cards, treat as client-side leak (server list already omits soft-unpublished for non-staff); fix via D2 filter, not a new list query.
- Revising SC-PACK-286 / SC-MAP-69 tests will fail until UI updated — expected; SC-PACK-165 ACL test currently asserts guest **hides** add-task-set — revise to CTA visible + gate (287).
- `checkGate` on task-set page also blocks unverified today — D8/SC-PACK-291 must skip gate on `view=live` for both anonymous and unverified, or view-only stays broken for email-unconfirmed users.
- Visual Spec colors for header may differ slightly from current Quasar header — apply within existing shell without inventing new theme system.

## Visual Spec (from mock)

Using mock: Maps list light + dark (Gemini assets in session / `.tmp-prepare-mock`). Surface: shared header + list hero chrome + card tops (card capacity chrome **not** in scope for MUST beyond noting conflict).

### Структура

1. Header: logo mark · section nav · theme · ВОЙТИ  
2. Hero: crumbs → title → subtitle; create on title band (guest: no «Все»)  
3. Fade illustration → page canvas  
4. Card grid (existing map/pack tiles)

### Copy / typography (product)

| Role | Copy (maps) | font-size approx | weight | line-height | Align |
|------|-------------|------------------|--------|-------------|-------|
| crumb | Лобби / Карты | 12–14 px | 400–500 | 1.2–1.4 | left |
| title | Карты | 32–40 px | 700+ | 1.1–1.25 | left |
| subtitle | Одобренные раскладки и ваши черновики | 13–15 px | 400 | 1.3–1.45 | left |
| create | create i18n + add icon | 12–14 px | 600 | 1 | center in btn |
| packs | catalogTitle / catalogSubtitle / create | same hierarchy | | | |

Nav/create UPPERCASE on mock — **optional**; prefer existing i18n casing unless trivial.

### Геометрия (approx @1280 CSS)

| Token | Value |
|-------|-------|
| header.height | ~56–64 px |
| page root pad (t/r/b/l) | 16–24 px (Quasar `q-pa-md` range OK) |
| crumb↔title | 8–14 px |
| title↔subtitle gap | 6–10 px |
| title band ↔ filter row (registered) | 12–20 px |
| title band ↔ card grid (guest, no filters) | 16–24 px |
| filter ↔ create gap (registered; same band on mock) | 8–12 px — **product keeps create on title band**, filters on row below (D7); mock «Все» beside create is out of scope for guest |
| create.height | 36–40 px (not dense; vs Quasar dense) |
| create.pad-x | 12–16 px |
| create.radius | 6–8 px |
| create.width | content (not full-bleed) |
| create.border-width | 0 (filled primary) |
| content side pad | 16–24 px |
| hero/title surface | full content width; no inset card; min-height n/a (content-driven) |

### Цвета

#### Light

| Token | Value | Confidence |
|-------|-------|------------|
| header.bg | `#d2e9fb`–`#d9eafb` | high |
| header.fg / nav / login | `#1c2e44`–`#2c3e50` | high–med |
| crumb.fg | `#5a8fb8`–`#6a9ec4` (muted blue over illustration) | med |
| title.fg | `#1e2e3e` / `#2c3e50` | high |
| subtitle.fg | muted slate ~`#5a6a7a` | med |
| page.canvas under cards | `#f7f4ec`–`#f9f6ee` | high |
| btn.create.bg | `$primary` `#1976d2` | med→canon |
| btn.create.fg | `#ffffff` | high |
| divider (hero→canvas) | n/a — soft illustration fade, not a 1px rule | — |
| shell bg | `pack-bg-*-light.jpg` | existing |

#### Dark

| Token | Value | Confidence |
|-------|-------|------------|
| header.bg | `#172236`–`#1a2237` | high |
| header.fg | `#9dafc5`–`#b3d4f3` | high–med |
| crumb.fg | `#8aa4bc`–`#a8c0d0` | med |
| title.fg | `#ffffff` | high |
| subtitle.fg | pale blue-grey ~`#a8c0d0` | med |
| page.canvas | `#141b2b`–`#151b2b` | high |
| btn.create.bg | `#1976d2` / `#1e88e5` | med |
| btn.create.fg | `#ffffff` | high |
| divider (hero→canvas) | n/a — soft illustration fade | — |
| shell bg | `pack-bg-*-dark.jpg` | existing |

### Icon ink / sizes

| Place | Size | Ink | Asset |
|-------|------|-----|-------|
| theme / login | ~20–24 | header.fg | Material |
| create + | ~18 | create.fg | Material `add` |
| brand mark | 40–48 | — | `src/assets/brand/logo.png` |

### Missing icons

- none

### Hover / focus

- hover не показан на макете — keep Quasar defaults; cards: no scale enlarge.

### Out of Visual Spec MUST

- Header wordmark text  
- Map card title/players pill «2-4» replacing capacity rows  
