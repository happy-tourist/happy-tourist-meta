## Context

See `proposal.md`. Runtime уже содержит unified lists … fan-out fix. Follow-up FIX: crumbs in page-container offset; moderation without «К наборам» + staff crumbs; narrow burger rightmost with Acc/Theme/Logout inside (only where section menu already exists).

Explore closed: D1–D21; crumbs overlap; moderation back; mobile burger Q1=Game unchanged, Q2=auth Theme stays / fold only with section menu.

## Goals / Non-Goals

**Goals**

- Prior goals — landed
- Breadcrumbs visible below elevated fixed header (page-container offset zone)
- Moderation chrome: no «К наборам» when crumbs cover path; staff crumbs Lobby / Модерация
- Narrow authenticated non-Game: burger **rightmost**; menu = sections (+ staff Модерация) then Acc / Theme / Logout; no Acc/Theme/Logout icons in toolbar on those screens

**Non-Goals**

- Changing Game header chrome for this FIX (no burger fold; Leave stays)
- Burger on auth/login; Theme remains in toolbar there
- Game header center pack title; tourist-room; new npm/SaaS; server payload for this FIX

## Decisions

### D1–D12, D14–D19 — Prior (unchanged summary)

As before. Runtime already landed for those decisions.

### D13 — Breadcrumbs below elevated header (revised again)

- **Choice:** Crumbs MUST render in the **page/layout zone that Quasar offsets for the fixed elevated header** — typically first child inside `q-page-container` (before `router-view`), **not** sibling between `</q-header>` and `<q-page-container>`. Vitest MUST assert crumbs inside page-container (offset zone), not only “not inside header”.
- **Why:** Crumbs were covered by elevated header after first D13 land.
- **Layers:** `App.vue`.

### D20 — Moderation chrome without «К наборам»; staff crumbs

- **Choice:** Pack moderation, staff queue/detail, author my-moderation: no `content.catalogNav` when App breadcrumbs cover Lobby / Packs / Модерация. Extend crumb routes for `content-staff`, `content-staff-request`, `content-staff-request-tasks`, `content-my-moderation` as applicable.
- **Why:** User: with crumbs, «К наборам» is redundant.
- **Layers:** `App.vue` crumbs; ContentStaff* / ContentPackModeration / ContentMyModeration.

### D21 — Narrow burger rightmost; fold Acc/Theme/Logout into menu

- **Choice:** When section navigation applies (`showSectionNav`: authenticated non-Game) **and** viewport is narrow: place the burger **rightmost** in the toolbar; put Packs / Maps / Support / staff Модерация **and** Account / Theme / Session logout **inside** that menu (order: sections, then Acc, Theme, Logout last). Do **not** also show Acc/Theme/Logout as separate toolbar controls on those screens. **Wide** viewports keep toolbar Acc/Theme/Logout with logout rightmost among that cluster (SC-BRAND-13). **Game:** do not introduce this fold — Leave (and Theme/Acc as today) stay in the toolbar; no section burger. **Auth/login:** no section burger; Theme stays in toolbar as today.
- **Why:** User Q1 Game unchanged; Q2 fold only where section menu already exists.
- **Layers:** `App.vue` toolbar / `q-menu`; vitest SC-BRAND-13 (wide) / 14 / 19.

## Risks / Trade-offs

- **[Risk] Dual view of same pack** → status per caller.
- **[Risk] Logo+right leave duplicate** → Accepted.
- **[Risk] Header crowding on mobile** → Burger right + fold Acc/Theme/Logout.
- **[Risk] Crumbs-only “outside header” tests miss overlay** → Assert page-container placement.
- **[Risk] SC-BRAND-13 DOM-order tests assume Acc/Theme/Logout always in toolbar** → Scope those assertions to wide / non-narrow.

## Migration Plan

1. Client: crumbs → page-container; staff crumbs; strip moderation catalogNav; narrow burger right + menu fold.
2. Vitest SC-BRAND-17…19, SC-PACK-193…195; lint/typecheck.
3. Rollback: revert deploy.

## Open Questions

None — Q1 Game unchanged; Q2 fold only with section menu; auth Theme stays.
