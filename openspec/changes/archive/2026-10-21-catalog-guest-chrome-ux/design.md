## Context

See `proposal.md` — Why. First apply shipped guest filter hide, list СНЯТО omit, add-task-set gate, live view-only, and Visual Spec title chrome. Follow-up (explore 2026-10-21): roll back create chrome to lobby parity; restore SC-PACK-285 translucent grid underlay (remove solid cream/navy); hide soft-unpublished **task sets** from non-staff on live pack detail (revise D6 / SC-PACK-132); translucent sticky `.game-hud` at alpha 0.78.

**Пакет:** client (`../happy-tourist.github.io`). Server create/add-task-set reject for anonymous — без изменений.

## Goals / Non-Goals

**Goals:**

- Hide all filter chrome for `auth.user?.anonymous` on packs + maps lists.
- Hide soft-unpublished (`hasLive && inCatalog === false`) cards from non-staff on those lists; staff keep СНЯТО chrome.
- Show «Добавить набор заданий» to anonymous and unverified on live pack → existing login/verify gate; keep view-only task-set entry (live boot skips auth gates).
- Title/subtitle band on packs/maps lists; logo without wordmark; map capacity rows.
- **Follow-up:** catalog create = Lobby Quasar `color="primary" icon="add"` (drop `unelevated` + `.catalog-list-create` custom sizing).
- **Follow-up:** remove `.catalog-list-page .pack-card-grid { background: solid }` so SC-PACK-285 translucent underlay shows through.
- **Follow-up:** on live pack detail, omit soft-unpublished task sets for `!isStaff` (staff keep muted «СНЯТО» + actions).
- **Follow-up:** `.game-hud` background `rgba(255,255,255,0.78)` / `rgba(18,18,18,0.78)` (not solid `#fff` / `#121212`).

**Non-Goals:**

- Wordmark in header; map-card «2-4» pill; top presence opaque panel; server list API / soft-unpublish storage rules; OAuth; prepare-mock as separate apply step.
- Mock create.height / pad / weight tokens — **superseded** by lobby parity (D9).

## Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| D1 | Guest = `anonymous` only for filter hide | Explore: «для всех гостей»; unverified keeps identity filters |
| D2 | Soft-unpublished omit on list: client defense-in-depth | Server list already staff-only; client filter `hasLive && inCatalog === false` for `!isStaff` |
| D3 | Reuse pack/map `gateOpen` / `gateMode` for add-task-set | Anonymous **and** unverified see CTA → `ensureEligible`; MUST NOT open editor |
| D4 | No header wordmark | Preserve SC-BRAND-01 |
| D5 | Keep map capacity rows | SC-MAP-55 |
| D6 | **REVISED:** Soft-unpublished task sets on live = staff-only | Explore: «снятые только стафу»; same as packs/maps lists; revises SC-PACK-132 (was muted card for non-staff) |
| D7 | Title/create layout: keep title+create row; hide filter row for guest | `v-if="!isGuest"` on filter row |
| D8 | Live view-only bypasses auth gates | Skip `checkGate` on `view=live` boot for anonymous + unverified |
| D9 | Catalog create = lobby primary parity | Explore: откат custom Visual Spec create; Quasar default primary like `LobbyPage` create |
| D10 | Catalog grid underlay = SC-PACK-285 translucent only | Remove solid `#f7f4ec` / `#141b2b` override on `.catalog-list-page .pack-card-grid` |
| D11 | Game HUD translucent alpha 0.78 | Explore: same as pack-card-grid; only sticky bottom `.game-hud` (not top presence row) |

### Точки врезки (client)

| Area | Files (indicative) |
|------|-------------------|
| Packs list filters + soft-unpub | `ContentCatalogPage.vue` |
| Maps list filters + soft-unpub | `MapsListPage.vue` |
| Add-task-set CTA + gate | `ContentPackPage.vue` |
| Task-set view-only | `ContentPackTasksPage.vue` (D8) |
| Soft-unpub task sets omit | `ContentPackPage.vue` — filter `live.taskSets` for `!isStaff` (`inCatalog === false`); staff keep muted + actions; ordinals among visible |
| Create rollback | `ContentCatalogPage.vue`, `MapsListPage.vue`, `app.scss` — drop `unelevated` / `.catalog-list-create` sizing overrides |
| Underlay restore | `app.scss` — delete solid `.catalog-list-page .pack-card-grid` rules |
| Game HUD | `GamePage.vue` scoped CSS `.game-hud` / `body.body--dark .game-hud` → rgba 0.78 |
| Tests | UnifiedList / ContentMaps / ContentPackAcl / SoftUnpublish / AppHeaderChrome / Game presence chrome as needed |
| Meta skills | content-pages, pack-cards, board/presence notes, SC-PACK-132 / SC-PRESENCE-34 |

Слои: pages → auth flags; soft-unpub predicate `inCatalog === false` (strict). Checklist — `tasks.md` (§7+ follow-up).

## Server contract (unchanged — reuse as-is)

Client-only; **no** new room messages, HTTP routes, schema, DB, auth API. Soft-unpublish **payload** may still include unpublished task sets in `getLivePack` for all JWT — client MUST omit for `!isStaff` (D6); do **not** require server filter for this change. Create/add-task-set reject for anonymous unchanged.

| Surface | Role |
|---------|------|
| `listCatalog` / `listMaps` | Soft-unpublished packs/maps only to staff (already) |
| `getLivePack` | May include soft-unpublished task sets; client D6 omit for non-staff |
| create / add-task-set | `requireVerifiedEditor` — no widen |
| Colyseus | Out of scope |

## Migration Plan

Client-only deploy. Rollback = revert client. No DB migration.

## Risks / Trade-offs

- Tests asserting muted «СНЯТО» for non-staff on pack detail (old SC-PACK-132) MUST be revised to omit — expected.
- Solid catalog canvas was verify-mock “fix”; product now prefers SC-PACK-285 — drop those solid assertions.
- HUD translucency may reduce contrast on budgets/strip — keep alpha 0.78; do not invent blur.

## Visual Spec (from mock) — still applies to title/header

Using mock: Maps list light + dark (`.tmp-prepare-mock`). **Create button geometry/weight from mock — superseded by D9 (lobby parity).** **page.canvas solid cream/navy under cards — superseded by D10 (SC-PACK-285).**

### Структура

1. Header: logo mark · section nav · theme · ВОЙТИ  
2. Hero: crumbs → title → subtitle; create on title band (guest: no «Все») — create = lobby primary  
3. Fade illustration → translucent card-grid underlay (not solid canvas)  
4. Card grid (existing map/pack tiles)

### Copy / typography (product)

| Role | Copy (maps) | font-size approx | weight | line-height | Align |
|------|-------------|------------------|--------|-------------|-------|
| crumb | Лобби / Карты | 12–14 px | 400–500 | 1.2–1.4 | left |
| title | Карты | 32–40 px | 700+ | 1.1–1.25 | left |
| subtitle | Одобренные раскладки и ваши черновики | 13–15 px | 400 | 1.3–1.45 | left |
| create | lobby Quasar primary (D9) | Quasar default | Quasar default | — | — |
| packs | catalogTitle / catalogSubtitle / create | same hierarchy | | | |

### Геометрия (approx @1280 CSS)

| Token | Value |
|-------|-------|
| header.height | ~56–64 px |
| page root pad | 16–24 px |
| crumb↔title | 8–14 px |
| title↔subtitle gap | 6–10 px |
| title band ↔ filter row (registered) | 12–20 px |
| title band ↔ card grid (guest) | 16–24 px |
| create.* | **lobby default** (D9 — not mock 36–40 / pad-x / radius table) |
| brand mark | 40–48 px |

### Цвета

#### Light

| Token | Value | Notes |
|-------|-------|-------|
| header.bg | `#d2e9fb`–`#d9eafb` | keep |
| header.fg | `#1c2e44`–`#2c3e50` | keep |
| crumb.fg | `#5a8fb8`–`#6a9ec4` | keep |
| title.fg | `#1e2e3e` | keep |
| subtitle.fg | `#5a6a7a` | keep |
| grid underlay | `rgba(255,255,255,0.78)` | D10 / SC-PACK-285 |
| game-hud | `rgba(255,255,255,0.78)` | D11 |
| btn.create | Quasar `$primary` | D9 |

#### Dark

| Token | Value | Notes |
|-------|-------|-------|
| header.bg | `#172236`–`#1a2237` | keep |
| header.fg | `#9dafc5`–`#b3d4f3` | keep |
| crumb.fg | `#8aa4bc`–`#a8c0d0` | keep |
| title.fg | `#ffffff` | keep |
| subtitle.fg | `#a8c0d0` | keep |
| grid underlay | `rgba(18,18,18,0.78)` | D10 |
| game-hud | `rgba(18,18,18,0.78)` | D11 |

### Out of Visual Spec MUST

- Header wordmark; map «2-4» pill; solid page.canvas under cards; custom create sizing class
