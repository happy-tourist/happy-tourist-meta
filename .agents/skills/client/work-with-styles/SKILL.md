---
name: work-with-styles
description: >-
  Use when adding, changing, reviewing, or debugging Vue 3 / Quasar 2 styles in
  the happy-tourist client: quasar.variables.scss theme tokens, global
  app.scss, page scoped CSS (especially GamePage board/cell/piece), Quasar
  utility classes in templates, Material Icons / Roboto extras, or color props
  on Quasar components.
---

# Work With Styles

Use this skill when working with styles in the **happy-tourist client**
(`happy-tourist.github.io`).

Stack: Vue 3 Composition API (`<script setup>`) + Quasar 2 + `@quasar/app-vite`
+ Sass/SCSS. Theming is Quasar Sass variables plus Quasar utility classes —
there is no Vuetify, no brand-profile color system, and no co-located
`styles.scss` next to components.

`quasar.config.ts` registers global CSS via `css: ['app.scss']` and loads
extras `roboto-font` + `material-icons`. Theme tokens live in
`src/css/quasar.variables.scss` (auto-available in `.vue` / `.scss` / `.sass`).

## Project Style Model

| Layer | Role | Location |
|-------|------|----------|
| Quasar theme Sass variables | Brand/palette tokens (`$primary`, `$negative`, …) | `src/css/quasar.variables.scss` |
| Global app CSS | App-wide rules (currently empty stub) | `src/css/app.scss` |
| Quasar extras | Roboto font + Material Icons | `quasar.config.ts` → `extras` |
| Page / component CSS | Scoped rules for custom UI (board, login card width) | `<style scoped>` in the `.vue` file |
| Template layout | Quasar utility / typography / color classes + props | Template `class` / `color` / `icon` |

There is no shared mixin/token SCSS pipeline beyond Quasar variables, and no
runtime theme override.

## Default Choices

- Prefer Quasar props (`color="primary"`, `color="negative"`, `flat`, `outline`,
  `dense`) so theme variables apply automatically.
- Prefer Quasar utility classes in templates for spacing, flex, typography, and
  text color (`q-pa-md`, `row`, `text-h5`, `text-grey-7`).
- Prefer Material Icons via Quasar `icon` / `q-icon` (`arrow_back`, `refresh`,
  `logout`, `sports_esports`, `visibility`) — already loaded as extras.
- Put game-board and other custom visuals in **scoped** `<style>` on the owning
  page (see `GamePage.vue`). Do not invent a co-located `styles.scss` folder
  pattern unless the project already has one for that feature.
- Keep `app.scss` for true globals only; do not dump page-specific board CSS
  there.
- Do not introduce Vuetify classes, brand profiles, or a second theme system.

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

These feed Quasar component colors and helpers such as `bg-negative`,
`text-white`, `color="primary"`. Change branding here first; avoid scattering
matching hex values across templates unless the UI is intentionally outside the
Quasar palette (checkers board wood tones, piece gradients).

## Global CSS

```scss
// src/css/app.scss
// app global css
```

File is registered in `quasar.config.ts` (`css: ['app.scss']`) but currently
has no rules. Add global resets / app shell styles here only when they must
apply outside a single page.

## Fonts And Icons

From `quasar.config.ts`:

```ts
extras: [
  'roboto-font',
  'material-icons',
],
```

- Typography defaults to Roboto via Quasar extras (not a custom font stack).
- Icons: Material Icons names as strings on `q-btn` `icon` / `q-icon` `name`.
- Do not add MDI / Font Awesome unless `extras` is updated on purpose; prefer
  Material Icons to match the existing lobby/login/game chrome.

## Scoped Page Styles

Custom CSS lives **inline in the SFC** with `scoped`, not in a sibling
`styles.scss`.

### Login card

```vue
<!-- LoginPage.vue -->
<style scoped>
.login-card {
  width: 100%;
  max-width: 400px;
}
</style>
```

Layout/spacing still uses Quasar classes (`q-pa-md`, `q-gutter-md`,
`flex flex-center`). Scoped CSS only constrains card width.

### Checkers board (`GamePage.vue`)

Board UI is custom (not Quasar widgets). Keep selectors local and class-driven:

| Class | Role |
|-------|------|
| `.game-header` | Cap header width (`max-width: 480px`) |
| `.board` | Wood-framed grid container; max width `min(90vw, 480px)` |
| `.board.disabled` | Slight opacity when `!game.canMove` |
| `.board-row` | 8-column CSS grid |
| `.cell` / `.light` / `.dark` | Square buttons; cream `#f0d9b5` / brown `#b58863` |
| `.cell.selected` | Yellow outline (`#ffeb3b`) for selected piece |
| `.cell.target::after` | Green move-hint dot |
| `.piece` / `.white` / `.black` / `.king` | Circular piece + king crown `♛` |

Template wires state via classes:

```html
<div class="board" :class="{ disabled: !game.canMove }">
  <button
    class="cell"
    :class="[
      (r + c) % 2 === 0 ? 'light' : 'dark',
      { selected: selected?.row === r && selected?.col === c },
      { target: isTarget(r, c) },
    ]"
  >
    <span v-if="cell" class="piece" :class="pieceClass(cell)" />
  </button>
</div>
```

When editing board visuals:

- Prefer adjusting existing classes over new global CSS.
- Keep move highlights as UI hints only (server remains source of truth).
- Preserve dark-square playable cells and aspect-ratio squares.
- Do not replace the board with Quasar grid components unless explicitly asked.

## Template Utilities And Color Props

Use Quasar helpers already present in login / lobby / game:

```html
<q-page class="q-pa-md flex flex-center column">
  <div class="row items-center justify-between q-mb-md">
    <div class="text-h5">Лобби</div>
    <div class="text-subtitle2 text-grey-7">…</div>
  </div>
  <q-btn color="primary" icon="sports_esports" label="Играть" />
  <q-banner dense rounded class="bg-negative text-white q-mb-md">…</q-banner>
</q-page>
```

Common groups in this app:

- Layout: `flex`, `flex-center`, `column`, `row`, `items-center`,
  `justify-between`, `full-width`, `col-12`, `col-sm-auto`
- Spacing: `q-pa-md`, `q-mb-md`, `q-mb-lg`, `q-mt-xs`, `q-gutter-md`,
  `q-gutter-sm`, `q-col-gutter-md`, `q-px-md`, `q-pb-md`
- Typography: `text-h5`, `text-subtitle1`, `text-subtitle2`, `text-caption`,
  `text-center`, `text-grey-7`
- Color / chrome: `color="primary"`, `color="grey-8"`, `bg-negative`,
  `text-white`, `rounded-borders`
- Feedback: `q-banner` with `dense` / `rounded` for store `error` strings

Prefer `color="primary"` / `bg-negative` over hardcoding `#1976d2` /
`#c10015` on Quasar components.

## Where Style Files Live

| Area | Typical path |
|------|----------------|
| Theme variables | `src/css/quasar.variables.scss` |
| Global CSS | `src/css/app.scss` |
| Extras / CSS registration | `quasar.config.ts` |
| Login layout tweak | `src/pages/LoginPage.vue` (`<style scoped>`) |
| Board / pieces | `src/pages/GamePage.vue` (`<style scoped>`) |
| Lobby / most chrome | Template Quasar classes only (`LobbyPage.vue`) |

Scaffold leftovers under `src/pages/index*` may still use Quasar demo classes;
prefer the login → lobby → game flow for new UI.

## How To Add Or Change Styles

1. Identify the surface: Quasar chrome (login/lobby) vs custom board (game).
2. For ordinary spacing, flex, type, and button colors — use Quasar classes /
   props first.
3. For theme-wide palette changes — edit `src/css/quasar.variables.scss`.
4. For true app-wide CSS — edit `src/css/app.scss` sparingly.
5. For board / piece / selection / target / disabled look — edit scoped CSS in
   `GamePage.vue`; keep class names (`board`, `cell`, `piece`, …) stable unless
   updating the template in the same change.
6. For small layout constraints (e.g. card max-width) — scoped class on the
   page, same pattern as `.login-card`.
7. Do not add Vuetify, brand profiles, or a design-token package for one-off
   needs.

## Common Mistakes

- Reaching for Vuetify utilities (`pa-8`, `d-flex`, `primary--text`) — this
  client is Quasar (`q-pa-md`, `row`, `text-primary` / `color="primary"`).
- Expecting brand profiles or runtime theme patches — none exist here.
- Moving board CSS into `app.scss` or a shared tokens file when scoped
  `GamePage` styles already own it.
- Hardcoding Quasar palette hex on `q-btn` / banners instead of
  `color="primary"` / `bg-negative`.
- Adding new icon packs while Material Icons extras already cover the UI.
- Restyling move highlights as authoritative rules — they are client hints;
  server validates moves.
- Creating co-located `styles.scss` folders by habit from other projects —
  this repo keeps page styles in the SFC.
