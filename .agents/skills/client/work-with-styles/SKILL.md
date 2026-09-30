---
name: work-with-styles
description: >-
  Use when adding, changing, reviewing, or debugging Vue 3 / Quasar 2 styles in
  the happy-tourist client: Quasar Dark, quasar.variables.scss, app.scss, scoped
  page CSS, Quasar utilities, Material Icons / Roboto. Core in SKILL.md; topic
  details in theme.md (Dark + /api/theme), board.md (GamePage tourist CSS),
  pack-cards.md (cascade yellow + answer/task tiles + catalog
  PackListCardTile + PackTaskSetCardTile summary chrome SC-PACK-229/239…246 +
  map list tiles).
---

# Work With Styles

Use this skill when working with styles in the **happy-tourist client**
(`happy-tourist.github.io`).

Stack: Vue 3 Composition API (`<script setup>`) + Quasar 2 + `@quasar/app-vite`
+ Sass/SCSS. Theming is Quasar Sass variables, Quasar utility classes, and the
built-in **Quasar `Dark` plugin** — no Vuetify, no brand-profile color system,
no co-located `styles.scss` next to components.

`quasar.config.ts` registers `css: ['app.scss']`, extras `roboto-font` +
`material-icons`, `framework.plugins: ['Dark']`, boots `theme` before `i18n` /
`colyseus`. Tokens: `src/css/quasar.variables.scss`.

## Specialized Topics

Read the matching file in this folder when the change involves that area
(do not load every file at once):

| Topic | File |
|-------|------|
| Quasar Dark + GET/POST `/api/theme` + header toggle wiring | [theme.md](theme.md) |
| Tourist board / presence / HUD / say CSS on GamePage | [board.md](board.md) |
| Cascade yellow + `.pack-card-grid` + answer/task + catalog/list + `PackTaskSetCardTile` + map tiles | [pack-cards.md](pack-cards.md) |

## Project Style Model

| Layer | Role | Location |
|-------|------|----------|
| Quasar Dark (runtime) | Chrome light / dark / device `auto` | See [theme.md](theme.md) |
| Brand logo / breadcrumbs CSS | ≥60px logo; `.app-breadcrumbs` inside `q-page-container` | `App.vue` scoped (+ `work-with-pages/shell.md`) |
| Quasar theme Sass variables | `$primary`, `$negative`, … | `src/css/quasar.variables.scss` |
| Global app CSS | `.text-muted`, `.pack-card-grid` | `src/css/app.scss` |
| Page / component CSS | Board, login card, tiles | `<style scoped>` / topic files |
| Template layout | Quasar utility / color props | Template `class` / `color` / `icon` |

Do **not** add a second theme engine — use Quasar `Dark` only.

## Default Choices

- Prefer Quasar props (`color="primary"`, `flat`, `dense`) and utility classes.
- Prefer Material Icons via `icon` / `q-icon` (already in extras). Game leave is
  the brand logo in `App.vue`, not a Material `logout` icon.
- Custom board / tile visuals → scoped SFC CSS (see topics). Do not invent
  co-located `styles.scss` folders.
- Keep `app.scss` for true globals only; do not dump board CSS there.
- Theme toggle lives in `App.vue` — do not duplicate per-page.

## Quasar Theme Variables

```scss
// src/css/quasar.variables.scss
$primary: #1976d2;
$secondary: #26a69a;
$accent: #9c27b0;
$dark: #1d1d1d;
$dark-page: #121212;
$positive: #21ba45;
$negative: #c10015;
$info: #31ccec;
$warning: #f2c037;
```

Change branding here first; board wood tones / piece gradients may stay outside
the Quasar palette on purpose.

## Global CSS

```scss
// src/css/app.scss
.text-muted { color: rgba(0, 0, 0, 0.54); }
.body--dark .text-muted { color: rgba(255, 255, 255, 0.7); }
```

Also hosts `.pack-card-grid` (see [pack-cards.md](pack-cards.md)).

## Fonts And Icons

`extras: ['roboto-font', 'material-icons']`. Do not add MDI / Font Awesome unless
`extras` is updated on purpose.

## Scoped Page Styles

Custom CSS lives **inline in the SFC** with `scoped`.

### Login card

```vue
<style scoped>
.login-card { width: 100%; max-width: 400px; }
</style>
```

### Content cascade / pack tiles / board

See [pack-cards.md](pack-cards.md) and [board.md](board.md).

## Template Utilities And Color Props

Prefer Quasar helpers already in login / lobby / game:

- Layout: `flex`, `flex-center`, `column`, `row`, `items-center`, `justify-between`
- Spacing: `q-pa-md`, `q-mb-md`, `q-gutter-md`, …
- Typography: `text-h5`, `text-subtitle2`, `text-muted` (prefer over `text-grey-7`)
- Color: `color="primary"`, `bg-negative`, `text-white`

## Where Style Files Live

| Area | Path |
|------|------|
| Dark plugin + boot | `quasar.config.ts`, `boot/theme.ts`, `stores/theme.ts` |
| Header toggle / logo / crumbs | `App.vue` |
| Variables / globals | `src/css/quasar.variables.scss`, `app.scss` |
| Board / pieces | `GamePage.vue` scoped → [board.md](board.md) |
| Pack/map tiles | tile components + [pack-cards.md](pack-cards.md) |

## How To Add Or Change Styles

1. Surface: Quasar chrome vs custom board vs pack tiles → pick topic.
2. Ordinary spacing/flex/type → Quasar classes; secondary labels → `text-muted`.
3. Light/dark chrome → [theme.md](theme.md).
4. Palette → `quasar.variables.scss`; globals → `app.scss` sparingly.
5. Board / tiles → scoped CSS; do not theme tourist tile fills for Dark.
6. Do not add Vuetify, brand profiles, or a design-token package for one-offs.

## Common Mistakes

- Vuetify utilities (`pa-8`, `d-flex`) — this client is Quasar.
- Second CSS theme system instead of Quasar `Dark`.
- Theme GET/POST from page templates; guest theme saved to server.
- JWT-only theme restore; `watch(() => […])` + replace `auth.user` after GET
  (SC-THEME-10) — see [theme.md](theme.md).
- `text-grey-7` for secondary chrome; retuning tile fills for Dark.
- Board CSS moved into `app.scss`; co-located `styles.scss` folders.
- `cascade-gap-outline` without warning outline; pack lists as `q-chip` rows;
  compose/`PackTaskTile` slots as dense `q-chip` instead of `.peek-slot-like`
  (see [pack-cards.md](pack-cards.md)).
  wrong tile sizes; `PackCsvImportDialog`; maps list as plain `q-item` —
  see [pack-cards.md](pack-cards.md).
