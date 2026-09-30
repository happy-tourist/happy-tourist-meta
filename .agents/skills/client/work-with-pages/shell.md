# App shell (`App.vue`)

Read with the [core pages skill](SKILL.md) when changing header, brand logo, breadcrumbs, section nav, Game leave/status, or theme banner wiring.


Every route renders inside:

```vue
<q-layout view="hHh lpR fFf">
  <q-header bordered>
    <q-toolbar>
      <!-- Brand logo left (≥60px); always same button (auth/other → lobby; lobby noop; Game leave) -->
      <button type="button" class="brand-logo-control"
        :aria-label="brandLogoAria" @click="onBrandLogoClick">
        <img :src="brandLogoUrl" alt="" class="brand-logo" />
      </button>
      <!-- Wide: Packs / Maps / Support (+ staff Модерация); Acc/Theme/Logout rightmost cluster -->
      <!-- Narrow + section nav: burger rightmost; sections + Acc/Theme/Logout inside menu (SC-BRAND-19) -->
      <q-space />
      <div v-if="isGameRoute" class="text-subtitle1 text-center">{{ statusLabel }}</div>
      <q-space />
      <!-- wide: account + theme; Game → header-game-leave; else session logout; narrow fold → burger -->
    </q-toolbar>
  </q-header>
  <!-- Game leave confirm dialog lives here (not on GamePage) -->
  <q-page-container>
    <!-- SC-BRAND-17 / SC-PACK-193 / SC-MAP-52: crumbs inside page-container offset zone (D13), not sibling above it -->
    <div v-if="breadcrumbItems.length" class="app-breadcrumbs" data-test-id="app-breadcrumbs">…</div>
    <q-banner v-if="theme.error" …>{{ theme.error }}</q-banner>
    <router-view />
  </q-page-container>
</q-layout>
```



Brand logo (`@/assets/brand/logo.png`, height ≥60px, `width: auto; object-fit: contain`) is always left as one interactive control. Click modes (`brandLogoMode`): **Game** → `leave` (`onExitClick` / confirm / `leaveGame`); **auth** routes (`login` / `forgot-password` / `confirm-email` / `reset-password`) → `toLobby` (`router.push({ name: 'lobby' })`; guest may bounce via `requiresAuth`); **lobby** → `noop` (same button, click ignored); other authenticated → `toLobby`. Aria: Game → `game.leave`; non-Game → `auth.backToLobby` (key kept for aria; no page «В лобби» buttons on Account/Support).

**Shared header chrome (SC-BRAND-11…20 / SC-LEAVE-08…12):** to the right of the logo on authenticated non-Game / non-auth **wide** screens — Packs / Maps / Support (+ staff «Модерация») + account + theme + **session logout** (logout rightmost). **Narrow** with section nav: burger **rightmost**; menu = sections then Acc / Theme / Logout (no separate toolbar Acc/Theme/Logout — SC-BRAND-14/19). On **Game** — centered match status, **right-side leave-from-room** (`header-game-leave`, same confirm as logo), **no** session logout, **no** section burger fold. Auth/login: Theme in toolbar; no section burger. Breadcrumbs sit **inside** `q-page-container` (header offset zone; `app-breadcrumbs` — SC-BRAND-17…18 / SC-PACK-193 / SC-MAP-52 / D13), not inside the elevated header and not as a sibling between `</q-header>` and `<q-page-container>`; staff/author moderation crumbs Lobby / Модерация (SC-BRAND-18); **Lobby** crumb always has `to: { name: 'lobby' }` and navigates on click (SC-BRAND-20 / D17 — modifier/middle-click keep native `<a :to>`); live pack/tasks and moderation chrome omit redundant «К наборам» / «Вернуться» / `content.catalogNav` when crumbs cover the path (SC-PACK-194/195). Shared theme toggle + `theme.error` banner live here (`useThemeStore`). Account nav + once-per-session email-verify reminder modal (registered, `needsEmailVerification`; mark `sessionStorage` seen **when shown**) also live in `App.vue` — CTA to `/account`; do not claim mail was already sent (`client-work-with-auth`). On **Game** (`route.name === 'game'`): centered status from `useGameStore` (same branches as former page `statusLabel` — SC-PRESENCE-23). Leave confirm when seated ∧ `playing` ∧ `finishPlace === 0` ∧ `!timeExpired`, else immediate `leaveGame` → lobby (`work-with-rooms`). Document title: `package.json` `productName` = `Happy Tourist` (`index.html` `<%= productName %>`); favicon: only `favicon.ico` in `index.html` / `public/` (no scaffold PNG icon set). Wire theme restore with a stable multi-source watch — `watch([() => auth.ready, () => auth.user?.id, () => auth.user?.anonymous], …)` → `syncFromAuthUser` (registered → `GET /api/theme`, guest → `localStorage`) — not JWT `user.theme` alone, and not `watch(() => […])` (new array each run). Do **not** replace `auth.user` after GET (theme lives in the theme store; SC-THEME-10). Do not duplicate a layout wrapper, per-page theme control, page «В лобби», Lobby section toolbar, or page-local leave/status header when adding pages.
