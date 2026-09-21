---
name: client-verify-code
description: >-
  Use when the user asks to verify code, check skill compliance, audit a branch
  diff vs master/main/merge-base, audit local diffs, or after implementing a
  change in the happy-tourist tourist Vue 3 client. Also when checking Vue
  Style Guide soft SFC order, or DRY / KISS / YAGNI balance on changed files.
  Reports three tiers: Violations, Warnings, Recommendations.
---

# Verify Code

Use this skill to check **all production code changed on the current branch** in
the `happy-tourist.github.io` client package against **all code-related** project
skills under `.agents/skills/client/`, plus the **built-in** checks in this file
(Vue Style Guide soft recommendations, Client conventions, and DRY / KISS /
YAGNI with conflict-aware judgment).

Default scope is the **full branch diff**: commits vs merge-base **and**
uncommitted working-tree changes. Do not stop at staged/unstaged/untracked.

This skill is primarily an **orchestrator**: domain rules live in the code
skills listed below — **read those skill files** when present and apply them; do
not restate or invent parallel domain rules here. Built-in sections below fill
gaps when a listed skill file is missing, and own Vue SFC order guidance plus
DRY / KISS / YAGNI balance.

**Paths:** skills currently live in **this repo** at `.agents/skills/client/`
(temporary; later move to **happy-tourist-meta**). Runtime code and `src/…`
paths are relative to **this client repo root** (`happy-tourist.github.io`).
Sibling Colyseus server is **`../happy-tourist-server`**, not a path under
skills.

**Stack assumptions:** Vue 3 Composition API / `<script setup>`, Pinia,
TypeScript, Quasar 2 + `@quasar/app-vite`, `@colyseus/sdk` (no axios), hash
router, GitHub Pages deploy.

## When To Use

- User asks to verify code / skill compliance / convention check
- After implementing a feature or refactor, before commit
- When reviewing the current branch (or a named path) for convention drift

## Scope: code skills only

Skills live under `.agents/skills/client/<name>/SKILL.md` in this repo
(temporary; later `happy-tourist-meta/.agents/skills/client/`).

### Always include (read and apply each when the file exists)

Verify against **every** skill in this set for the checked code — not a subset
guessed from the path. Path routing below only helps prioritize deeper reading;
it does **not** allow skipping skills from this list.

| Skill | Concern |
|-------|---------|
| `colyseus-client` | `client.http`, room messages, Colyseus Client |
| `client-work-with-errors` | store error + `q-banner`, room `onError` |
| `work-with-stores` | Pinia auth/theme/game |
| `work-with-forms` | LoginPage `q-form` |
| `work-with-pages` | routes + guards; App theme shell watch |
| `client-work-with-structure` | pages/boot/stores layout |
| `work-with-styles` | Quasar Dark + `/api/theme` + tourist board CSS |
| `client-work-with-auth` | Colyseus Auth |
| `work-with-localization` | vue-i18n boot |
| `work-with-lobby` | live LobbyRoom subscribe / leave before enter / create/join |
| `work-with-rooms` | Room lifecycle |
| `work-with-game-board` | Tourist board + seat pieces / strip + turn select/hints/`sendMove` |
| `work-with-env-deploy` | `VITE_*`, hash router, GH Pages |

If a new code skill appears under `.agents/skills/client/` (same kind: how to
write app code), include it too. Prefer reading one extra skill over missing a
rule.

If a listed skill file is **missing**, do not invent a parallel rulebook — apply
**Built-in: Client conventions** for that concern and continue.

### Always exclude

Do **not** use these for client-verify-code (unless the user explicitly asks):

| Skill | Why excluded |
|-------|----------------|
| `client-locate-change-points` | planning where to edit |
| `client-verify-code` | this orchestrator |
| `client-align-code` | alignment / docs authorship |
| `openspec` / opsx skills | change workflow / specs |
| `commit` | commit messages |

Skip verifying files that are **only** tests (`**/tests/**`, `**/*.spec.ts`,
`**/*.spec.js`, `**/__tests__/**`) unless the user explicitly asks to include
them. Focus on production/source code under `src/`.

## What To Check

**Always** inspect the full branch change set. Working tree alone is not enough.

Run git from the **repository root** (this client package is the git root).

1. Resolve merge-base: `git merge-base HEAD <base-ref>`. Prefer `origin/main`
   (this repo), else `origin/master`, `main`, `master`, `develop` — first that
   exists.
2. Committed on the branch: `git diff --name-only <merge-base>...HEAD` and
   `git diff <merge-base>...HEAD`
3. Staged: `git diff --cached --name-only` and `git diff --cached`
4. Unstaged: `git diff --name-only` and `git diff`
5. Untracked: `git ls-files --others --exclude-standard` (read those files fully)

Union of 2–5 is the file set (filter to client production sources unless the user
widened scope). Never skip step 2 because the working tree looks small.

Narrow only when the user **explicitly** asks for working-tree-only / only staged,
or names a **folder or path**.

Skip unrelated noise (lockfiles, coverage, binary assets, deploy blobs) unless
the user asked to include them.

## Commands (before done)

Run package scripts from the **client package root** (`happy-tourist.github.io`).
Fix failures before claiming done. Do not skip lint / typecheck after changes
unless the user explicitly asked for a report-only pass with no tooling.

When verification is part of an implementation you own, or when the user asked
to verify-and-fix / “make sure it passes”, run:

| Command | Purpose |
|---------|---------|
| `npm run lint` | ESLint (flat config) |
| `npm run typecheck` | `vue-tsc` / typecheck |
| `npm run build` / `quasar build` | Production SPA build |

Prefer `npm run lint` and `npm run typecheck` for touched areas. Run
`npm run build` / `quasar build` when the user asks for build parity or deploy
confidence. For interactive smoke, you may start `quasar dev` when needed;
prefer finite gate commands (lint/typecheck/build). Do not treat passing
lint/typecheck/build as a substitute for skill checks.

Report failed lint/typecheck/build output under **Violations** (tooling) with
the command and a short failure summary. Do not invent eslint rules beyond the
project config and skill text.

## Workflow

1. Collect the file set: merge-base…HEAD **plus** staged, unstaged, untracked
   (or the path the user named). Exclude test-only files per Scope.
2. **Read all skills from the Always include table** that exist on disk (do not
   rely on memory; do not skip because the path “looks unrelated”).
3. For each source file, apply every code skill whose rules can touch that file.
   When unsure, apply the skill. If missing, use Built-in Client conventions.
4. For every checked `.vue` file, apply **Built-in: Vue Style Guide**.
5. Apply **Built-in: Client conventions** to all checked production files.
6. Apply **Built-in: DRY / KISS / YAGNI** to all checked production files.
   Resolve principle conflicts with the order in that section **before**
   classifying or fixing — never emit opposing principle fixes for the same hunk.
7. Classify every finding into Violations / Warnings / Recommendations
   (see Severity). Each item: file path, what is wrong, which skill/rule, and
   the expected pattern. Soft Prefer guidance → Warnings or Recommendations,
   not silence.
8. Fix **Violations** when the user asked to verify-and-fix, or when
   verification runs as part of an implementation task you own. Fix **Warnings**
   on verify-and-fix when the preferred pattern clearly fits. Fix
   **Recommendations** only if the user asked to tidy / apply soft order.
   On principle fixes, obey **Built-in: DRY / KISS / YAGNI** conflict order so
   DRY does not fight KISS/YAGNI (and vice versa). Otherwise list findings and
   wait.
9. When implementing / verify-and-fix: **run** `npm run lint` and
   `npm run typecheck` (and `npm run build` / `quasar build` if requested) from
   the client package root; fix failures before claiming done.

Do not praise compliant code. Do not turn this into a product/bug review skill.

## Built-in: Vue Style Guide

Source: [Vue.js Style Guide — Priority C Rules: Recommended](https://vuejs.org/style-guide/rules-recommended.html),
adapted for **Composition API / `<script setup>`** (this package does not use
Options API category order). Soft guidance on changed/new `.vue` files. Report as
`(skill: client-verify-code / Vue Style Guide)`.

**Soft recommendations only — tidy order, do not enforce rigidly.**

- Goal: keep SFCs readable with a consistent block order.
- **Not** a hard lint: do not demand a full reorder if it risks breakage.
- Prefer clear, safe tidy-ups over micro-nits.
- Severity: always **Recommendations** — unless verify-and-fix and reorder is
  clearly safe.

### SFC top-level block order

In this Vue 3 / `<script setup>` project prefer:

`<template>` → `<script setup>` → `<style>` (style last).

Also acceptable: `<script setup>` → `<template>` → `<style>`.

**Suggest fix when:** `<style>` is before `<script setup>`/`<template>`, or both
block-order variants appear in the same change set without reason.

Do **not** apply Options API component-options category order (`data` /
`computed` / `methods` / lifecycle hooks as option keys) — this package uses
Composition API inside `<script setup>`.

### Element attribute order

On elements/components in templates, prefer:

1. `is`
2. `v-for`
3. `v-if` / `v-else-if` / `v-else` / `v-show` / `v-cloak`
4. `v-pre` / `v-once`
5. `id`
6. `ref` / `key`
7. `v-model`
8. Other attributes (bound and unbound)
9. `v-on` / `@…`
10. `v-html` / `v-text`

**Suggest** only obvious inversions (e.g. `@click` before `v-if`). Do not
rewrite every attribute line for minor reordering unless asked to fix.

### Empty lines between multi-line declarations

When multi-line reactive blocks / adjacent multi-line declarations become hard
to skim, prefer a blank line between them. Do not flag readable dense
single-line blocks.

## Built-in: Client conventions

Grounded in `AGENTS.md`. Report as `(skill: client-verify-code / Client conventions)`.
Apply always; sibling skills win when they exist and conflict on a detail.

### Language and architecture

- **TypeScript** + **Vue 3** SFCs with **Composition API** / `<script setup>`.
- **Pinia** for application state (`stores/auth`, `stores/theme`, `stores/game`); do not add a
  parallel Vuex or ad-hoc global reactive singleton for game/auth/theme.
- **Quasar 2** components and theming (`q-page`, `q-card`, `q-btn`, `q-form`,
  `q-banner`, …); prefer Quasar patterns already used on Login/Lobby/Game pages.
- Dependency direction: `pages` → `stores` / `boot` / `components`. Keep
  Colyseus I/O inside Pinia stores (`auth`, `theme`, `game`) rather than scattering
  `client.*` calls across many components.
- Prefer the login / lobby / game flow over scaffold leftovers
  (`EssentialLink.vue`, `example-store.ts`).

### Colyseus / HTTP (no axios)

- Shared `Client` from `src/boot/colyseus.ts`
  (`new Client(import.meta.env.VITE_COLYSEUS_URL)`). Prefer importing `client`
  from `@/boot/colyseus` in script (not only `$colyseus`).
- **No axios layer** — auth and rooms go through the Colyseus SDK client.
- Lobby listing: `client.http.get('/rooms/tourist')` (not removed
  `getAvailableRooms`).
- Room type constant `TOURIST_ROOM = 'tourist'` in `stores/game`.
- Room lifecycle: `create` / `joinById` / `joinOrCreate`, `onStateChange`,
  `leave`, `sendMove` → `room.send('move', { side, row, col })`. `_enterRoom`
  must `_attachRoom` immediately after `connect()` — no `await` (e.g.
  `unsubscribeLobby`) before the listener, or the first `ROOM_STATE` is missed.
- GamePage: tourist board with synced seat pieces; top presence (opponents /
  spectator all) + sticky seated `.game-hud` (own + strip row N,E,W,S / narrow
  HUD ≤~420 → 2×2 — **no** chip/`q-menu`); dual rings from
  `turnUntil` / `reconnectUntil` around 72px avatar in 96px chrome; avatar as
  sibling `<img class="presence-avatar">` on top — not only in `q-circular-progress`
  default slot without `show-value` (SC-PRESENCE-12); board `--gap`/`--radius` 2;
  local selection / hints when `isMyTurn` and eligible; travel + return-from-nearest-center
  animation; finish 2×2 → nearest legal center; submit only via `game.sendMove`.
  Brand logo + match status in App header; Game leave via logo (not Material `logout`, not page-local).
  Return: confirm modal → same red `.tile--target` as move; `GRILLE_ANIM_MS = 1000`
  on board + strip chrome.
- Mirror `currentTurnSessionId` / `turnUntil` / `turnBudgetSeconds` / seat `timeExpired`;
  do not invent alternate move/turn shapes.

### Auth and routing

- Auth via `client.auth` (Colyseus Auth): register, email/password, anonymous,
  sign-out — through `stores/auth`. Token key `colyseus-auth-token`.
- Router: **hash** mode (`/#/lobby`, `/#/game/...`). Guards await
  `auth.whenReady()`, then enforce `requiresAuth` / `guest` meta.
- Do not switch to history mode without an explicit request (GitHub Pages has
  no history fallback; CI copies `index.html` → `404.html`).

### Errors and async

- Auth/game actions catch errors into store `error` string; pages show
  `q-banner`. Theme save/restore failures use `theme.error` + `App.vue` banner.
- Do not swallow errors with empty `catch` (except established room-leave
  closed-room swallow in `leaveGame`).
- `GamePage` calls `rejoinGame(roomId)` if Pinia lost the room (token → `joinById`); failed rejoin → lobby.
- Theme restore: stable multi-source `watch` in `App.vue`; after GET keep theme
  in `theme` store — do not replace `auth.user` (SC-THEME-10 request storm).

### Env and deploy

- Vite env: `VITE_COLYSEUS_URL`, `VITE_API_URL` (typed in `env.d.ts`; local
  `.env.development` / `.env.production`; CI injects GitHub Actions `vars`).
- Do not hardcode production host URLs in app logic when env already covers them.
- Deploy: `.github/workflows/deploy.yml` (`quasar build -m spa`).

### UI / i18n / styles

- Chrome Dark: Quasar `Dark` + `boot/theme` + `stores/theme` + `App.vue` header;
  guest `localStorage` (`ht-theme`); registered `GET`/`POST` `/api/theme`.
- Theme Sass in `src/css/quasar.variables.scss`; global styles in
  `src/css/app.scss`.
- i18n via boot `i18n` + `vue-i18n` (default locale `en-US`).
- Path alias `@/*` → `src/*`.

### Severity cues for built-in conventions

| Tier | Examples |
|------|----------|
| **Violations** | Ad-hoc axios / second HTTP client; Colyseus I/O scattered outside stores; empty `catch` on auth/game; history router without request; inventing game rules / board authority on client |
| **Warnings** | Importing only `$colyseus` when `@/boot/colyseus` fits; wiring LobbyPage back to HTTP `refreshRooms` poll; hardcoding env URLs; expanding scaffold leftovers instead of login/lobby/game |
| **Recommendations** | Vue SFC block order; minor formatting / import tidy; mild DRY/KISS polish |

## Built-in: DRY / KISS / YAGNI

Report as `(skill: client-verify-code / DRY-KISS-YAGNI)`. Apply to all checked
production files. Principles are **judgment lenses**, not absolute mandates.

**Core:** Prefer the simplest correct code that meets the real requirement.
Project skills and Client conventions **win** over DRY, KISS, and YAGNI. When
principles pull opposite ways, choose **one** outcome using the conflict order
below — never “satisfy DRY” by violating KISS/YAGNI or a domain skill, and never
“simplify” by deleting a required shared contract.

### Verify lens

| Principle | Flag when | Do not flag when |
|-----------|-----------|------------------|
| **DRY** | Same *knowledge/rule* is duplicated in the change set and will drift if only one side changes | Similar-looking code with different domain meaning; intentional parallel helpers a skill requires; trivial short copies that stay clearer separate |
| **KISS** | New indirection, factory, or clever layer that obscures the change without payoff | Required layering from skills (Pinia stores, Colyseus I/O placement, Quasar patterns) |
| **YAGNI** | Abstraction, extension point, or helper built for hypothetical future call sites (0–1 real uses of that shape) | Small helper with 2+ real call sites of the *same* shape already in the change |

### Conflict resolution (required before classify / fix)

Apply **in order**:

1. **Domain skills / Client conventions / contracts** — do not dedupe or simplify away a required pattern (e.g. keep store-private leave splits when `work-with-rooms` says so).
2. **YAGNI** — remove or do not introduce unused / future-only abstractions.
3. **KISS** — prefer direct code over shared machinery when duplication is small or meanings differ.
4. **DRY** — extract only when identical knowledge would otherwise drift; the extraction must stay simple.

**Anti-conflict rule:** Do not report both “extract for DRY” and “inline for KISS/YAGNI” on the same hunk. Pick one using the order above. On verify-and-fix, apply that single outcome. Prefer **one finding per hunk** that names the winning principle (mention secondary principles only as supporting reason in the same bullet).

### Severity cues

| Tier | When |
|------|------|
| **Violations** | Rare for pure principles. Only if a new premature abstraction **breaks** a required skill/contract, or critical contract knowledge is duplicated **and already diverges** in the same change set |
| **Warnings** | Identical-knowledge duplication likely to drift; unused/future-only abstraction; complexity that obscures a preferred skill pattern |
| **Recommendations** | Mild duplication or slightly overcomplicated local code where a simpler safe shape is obvious |

### Red flags — STOP and re-resolve

- “Merge for DRY” when a skill says keep the split
- Shared util with one call site “for later” (YAGNI)
- Inlining a module that multiple real callers already need (false KISS)
- Opposing principle findings on the same hunk without conflict resolution

## Path hints (optional prioritization only)

Use only to decide **where to look harder**, never to drop a skill from Always include.

| Change / path signals | Look harder at |
|----------------------|----------------|
| `src/boot/colyseus.ts`, `client.http`, room `send` / `onStateChange` | `colyseus-client`, `work-with-rooms` |
| `stores/auth.ts`, LoginPage auth flows | `client-work-with-auth`, `work-with-forms` |
| `stores/theme.ts`, `App.vue` theme watch / toggle | `work-with-styles`, `work-with-stores`, `work-with-pages` |
| `stores/game.ts`, LobbyPage | `work-with-lobby`, `work-with-stores` |
| `GamePage`, board CSS / move UI | `work-with-game-board`, `work-with-styles` |
| `src/router/**`, route meta / guards | `work-with-pages` |
| `src/pages/**`, `src/boot/**`, `src/stores/**` | `client-work-with-structure` |
| store `error`, `q-banner`, room `onError` | `client-work-with-errors` |
| `src/i18n/**`, boot `i18n` | `work-with-localization` |
| `.env*`, `env.d.ts`, `quasar.config.ts`, `.github/workflows/**` | `work-with-env-deploy` |
| `src/css/**`, Quasar variables, board styles, Dark boot | `work-with-styles` |
| `*.vue` block/attribute order | Vue Style Guide |

## High-signal checks (examples, not a full rulebook)

Reminders to **open the skill** (or Built-in) — skill text wins.

- **Colyseus**: shared Client from boot; HTTP via `client.http`; Game `room.send`
  only when rules exist; I/O in Pinia stores.
- **Auth**: `client.auth` + `stores/auth`; guards wait for `whenReady()`.
- **Theme**: Quasar Dark + `stores/theme`; registered GET restore ≠ JWT-only;
  stable App `watch([() => ready, () => id, () => anonymous])`; no `auth.user`
  replace after GET (SC-THEME-10).
- **Lobby/rooms**: live LobbyRoom `subscribeLobby`; create/join/leave through `stores/game`.
- **Board**: tourist CSS Grid on GamePage + pieces from seats; no `canMove` / draughts cells.
- **Errors**: store `error` + `q-banner` (theme → `App.vue`); no empty `catch`.
- **Env/deploy**: `VITE_*`, hash router, GH Pages SPA build.
- **Vue SFC order** (soft): `<template>` → `<script setup>` → `<style>`.
- **DRY/KISS/YAGNI**: conflict order — skills → YAGNI → KISS → DRY; one fix per hunk.

## Severity

Map each skill finding by how the source skill phrases the rule. Skill text wins;
when wording mixes levels, pick the strongest that still applies.

| Tier | When | Skill wording cues (examples) |
|------|------|-------------------------------|
| **Violations** | Hard break of a required pattern / contract | Do / Don't, Must, Never, Always, Hard Rules, forbidden empty `catch`, axios instead of Colyseus client, Colyseus I/O outside stores |
| **Warnings** | Preferred pattern clearly fits; allowed exception does **not** apply | Prefer, Should, “use X instead of Y” when X fits |
| **Recommendations** | Soft tidy / style / optional polish | usually, Soft, Vue Style Guide, optional SFC block reorder, mild DRY/KISS |

Do not invent findings outside the code skills and the built-in sections above.
Do not inflate Prefer into Violations.

## Output

Report language: **English** for prose; keep paths, symbols, and code as-is.

Always include all three finding sections. If a section has no items, put a
single line `- none` (do not mix items and `none` in the same section).

Format:

```text
## Verify Code

Scope: branch vs <merge-base> (<base-ref>) + staged + unstaged + untracked [+ path if used]
Skills: all code skills (excl. locate/align/openspec/commit) + Vue Style Guide + Client conventions + DRY/KISS/YAGNI
Lint/typecheck/build: <commands run and pass/fail summary, or skipped with reason>

### Violations
- `path` — <what is wrong> (skill: `<name>`)
  Expected: <short correct pattern or cite skill section>

### Warnings
- `path` — <what is wrong> (skill: `<name>`)
  Expected: <short correct pattern or cite skill section>

### Recommendations
- none
```

Empty-section example: only `- none` under that heading. Item shape is the same
in every tier. Do not collapse Warnings/Recommendations into Violations or omit
soft Prefer findings.

## Red Flags — collect the branch first

These mean STOP and re-collect files vs merge-base before reporting:

- Ran only `git diff` / `--cached` / untracked
- “User didn’t ask for the whole branch”
- “Working tree has the real work”
- “I’ll check working tree first, branch later”
- Report Scope without `branch vs <merge-base>`

## Related Skills

All skills in the Always include table. Vue Style Guide soft order, Client
conventions, and DRY / KISS / YAGNI balance are owned by this skill when sibling
files are absent.
