# App shell (`App.vue`)

Read with the [core pages skill](SKILL.md) when changing header, brand logo, breadcrumbs, section nav, Game leave/status, or theme banner wiring.


Every route renders inside:

```vue
<q-layout view="hHh lpR fFf" class="app-page-bg">
  <q-header bordered>
    <q-toolbar>
      <!-- Brand logo left (≥60px); always same button (auth/other → lobby; lobby noop; Game leave) -->
      <button type="button" class="brand-logo-control"
        :aria-label="brandLogoAria" @click="onBrandLogoClick">
        <img :src="brandLogoUrl" alt="" class="brand-logo" />
      </button>
      <!-- Wide: Packs / Maps / Support (+ staff Модерация); Acc/Theme + session control rightmost -->
      <!-- registered: Acc + Theme + Logout; anonymous: Theme + login (no Acc); auth screens: Theme only -->
      <!-- Narrow + section nav: burger rightmost; sections + cluster inside menu (SC-BRAND-19/22/24) -->
      <q-space />
      <div v-if="isGameRoute" class="text-subtitle1 text-center">{{ statusLabel }}</div>
      <q-space />
      <!-- wide: account (registered only) + theme; Game → header-game-leave;
           else registered logout / guest login icon; AUTH_ROUTE_NAMES → neither; narrow → burger -->
    </q-toolbar>
  </q-header>
  <!-- Game leave confirm dialog lives here (not on GamePage) -->
  <!-- SC-BRAND-23 / D7: --with-crumbs flex column so Quasar page min-height does not stack under crumbs -->
  <q-page-container :class="{ 'q-page-container--with-crumbs': breadcrumbItems.length > 0 }">
    <!-- SC-BRAND-17 / SC-PACK-193 / SC-MAP-52: crumbs inside page-container offset zone (D13), not sibling above it -->
    <div v-if="breadcrumbItems.length" class="app-breadcrumbs" data-test-id="app-breadcrumbs">…</div>
    <q-banner v-if="theme.error" …>{{ theme.error }}</q-banner>
    <router-view />
  </q-page-container>
</q-layout>
```



**Shell page background (SC-BRAND-21):** `class="app-page-bg"` on `q-layout` (all routes). CSS in `src/css/app.scss`: fixed `cover` from `src/assets/packs/pack-bg-{desktop|mobile}-{light|dark}.jpg`; dark via `body.body--dark`; mobile `@media (max-width: 599px)` (Quasar `sm` − 1, independent of header burger `md`); transparent page canvas (`.q-page` / `.q-page-container` / `.app-breadcrumbs` / `--q-page-bg`) so art shows through. Elevated header/toolbar MAY stay opaque. Do **not** add per-page background classes. Card-grid underlay: see [pack-cards.md](../work-with-styles/pack-cards.md) (SC-PACK-285). Game board holes still show page bg (SC-THEME-07).

Brand logo (`@/assets/brand/logo.png`, height ≥60px, `width: auto; object-fit: contain`) is always left as one interactive control. Click modes (`brandLogoMode`): **Game** → `leave` (`onExitClick` / confirm / `leaveGame`); **auth** routes (`login` / `forgot-password` / `confirm-email` / `reset-password`) → `toLobby` (`router.push({ name: 'lobby' })`; guest may bounce via `requiresAuth`); **lobby** → `noop` (same button, click ignored); other authenticated → `toLobby`. Aria: Game → `game.leave`; non-Game → `auth.backToLobby` (key kept for aria; no page «В лобби» buttons on Account/Support).

**Shared header chrome (SC-BRAND-11…20 / 22 / 24 / SC-LEAVE-08…12):** to the right of the logo on authenticated non-Game / non-auth **wide** screens — Packs / Maps / Support (+ staff «Модерация») + account-cluster. **Registered:** account + theme + **session logout** (logout rightmost — SC-BRAND-13). **Anonymous guest:** theme + **login** icon (same flat/dense pattern as logout; aria `auth.login` / «Войти»); account **omitted** (cabinet registered-only); click → `{ name: 'login', query: { redirect: fullPath } }` without logout (SC-BRAND-22; design D4). **Narrow** with section nav: burger **rightmost**; menu = sections then the same cluster (registered Acc/Theme/Logout; guest Theme/login last — SC-BRAND-14/19/24; no separate toolbar cluster). On **Game** — centered match status, **right-side leave-from-room** (`header-game-leave`, same confirm as logo), **no** session logout **and no** guest login icon, **no** section burger fold. On `AUTH_ROUTE_NAMES` (`login` | `forgot-password` | `confirm-email` | `reset-password`): Theme in toolbar; **neither** logout nor guest login-icon; no section burger. Breadcrumbs sit **inside** `q-page-container` (header offset zone; `app-breadcrumbs` — SC-BRAND-17…18 / SC-PACK-193 / SC-MAP-52 / D13), not inside the elevated header and not as a sibling between `</q-header>` and `<q-page-container>`; when crumbs are present, page min-height accounts for the strip so short packs/maps content does not force phantom scroll (SC-BRAND-23 / design D7); staff/author moderation crumbs Lobby / Модерация (SC-BRAND-18); **Lobby** crumb always has `to: { name: 'lobby' }` and navigates on click (SC-BRAND-20 / D17 — modifier/middle-click keep native `<a :to>`); live pack/tasks and moderation chrome omit redundant «К наборам» / «Вернуться» / `content.catalogNav` when crumbs cover the path (SC-PACK-194/195). Shared theme toggle + `theme.error` banner live here (`useThemeStore`). Account nav + once-per-session email-verify reminder modal (registered, `needsEmailVerification`; mark `sessionStorage` seen **when shown**) also live in `App.vue` — CTA to `/account`; do not claim mail was already sent (`client-work-with-auth`). On **Game** (`route.name === 'game'`): centered status from `useGameStore` (same branches as former page `statusLabel` — SC-PRESENCE-23). Leave confirm when seated ∧ `playing` ∧ `finishPlace === 0` ∧ `!timeExpired`, else immediate `leaveGame` → lobby (`work-with-rooms`). Document title: `package.json` `productName` = `Happy Tourist` (`index.html` `<%= productName %>`); favicon: only `favicon.ico` in `index.html` / `public/` (no scaffold PNG icon set). Wire theme restore with a stable multi-source watch — `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `syncFromAuthUser` (registered → `GET /api/theme`, guest → `localStorage`) — not JWT `user.theme` alone, and not `watch(() => […])` (new array each run). Do **not** replace `auth.user` after GET (theme lives in the theme store; SC-THEME-10). Do not duplicate a layout wrapper, per-page theme control, page «В лобби», Lobby section toolbar, or page-local leave/status header when adding pages.
