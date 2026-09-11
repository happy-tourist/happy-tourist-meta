---
name: work-with-env-deploy
description: >-
  Use when changing Vite env vars, GitHub Pages deploy, hash router mode, or
  CI build for the happy-tourist client — VITE_COLYSEUS_URL, VITE_API_URL,
  env.d.ts, .env.development / .env.production, quasar.config.ts publicPath /
  vueRouterMode, or .github/workflows/deploy.yml.
---

# Work With Env And Deploy

Use this skill for **env vars**, **hash routing**, and **GitHub Pages deploy** of the Vue 3 Quasar checkers client (`happy-tourist.github.io`).

Deploy target: GitHub Pages **user/org site at domain root**. Router mode is **hash** so deep links work without a history fallback.

## Hard rules

| Do | Don't |
|----|--------|
| Keep `vueRouterMode: 'hash'` for Pages | Switch to `history` without a real history fallback on the host |
| Keep `publicPath: '/'` (domain root) | Assume project-site `base` like `/repo-name/` |
| Type new `VITE_*` keys in `env.d.ts` | Read secrets from the client; these are build-time public URLs |
| Inject prod URLs via GitHub Actions `vars` in CI | Commit production secrets into the repo |
| Run `npm` / `quasar` lint / typecheck / build from client package root | Skip verification or assume pass without running |

## Env vars

| Variable | Role | Local (`.env.development`) | Prod (`.env.production`) |
|----------|------|----------------------------|--------------------------|
| `VITE_COLYSEUS_URL` | Colyseus SDK / WebSocket (`new Client(...)` in `src/boot/colyseus.ts`) | `ws://localhost:2567` | `wss://happy-tourist.duckdns.org` |
| `VITE_API_URL` | HTTP base (same host family as Colyseus) | `http://localhost:2567` | `https://happy-tourist.duckdns.org` |

- Typed in `env.d.ts` as `ImportMetaEnv` (`readonly VITE_COLYSEUS_URL`, `readonly VITE_API_URL`).
- Consumed at build/dev time via `import.meta.env.VITE_*` (Vite).
- Local defaults: **localhost:2567**. Prod defaults: **duckdns WSS/HTTPS**.
- Sibling server: `../happy-tourist-server` (same host/port locally).

When adding a new env key:

1. Add to `.env.development` and `.env.production`.
2. Extend `ImportMetaEnv` in `env.d.ts`.
3. Wire CI `env:` in `.github/workflows/deploy.yml` from `vars.*` if the production build needs it.

## Hash router

Required for GitHub Pages **without** a history fallback (deep links must not hit the server as real paths).

| File | Setting |
|------|---------|
| `quasar.config.ts` | `vueRouterMode: 'hash'`; `publicPath: '/'` |
| `src/router/index.ts` | Uses `createWebHashHistory` when `QUASAR_VUE_ROUTER_MODE` is not `'history'` |

URLs look like `/#/lobby`, `/#/game/:roomId`. Do not flip to history mode for Pages unless the host rewrites all routes to `index.html`.

## GitHub Pages deploy

Workflow: `.github/workflows/deploy.yml` (`Deploy Client`).

Triggers: `push` to `master`, `workflow_dispatch`.

### Build steps (must stay in sync)

1. `npm ci` (Node 22).
2. **CI injects GitHub Actions `vars`**:
   - `VITE_COLYSEUS_URL: ${{ vars.VITE_COLYSEUS_URL }}`
   - `VITE_API_URL: ${{ vars.VITE_API_URL }}`
3. `npx quasar build -m spa`
4. Prepare artifact:
   - copy `dist/spa/index.html` → `dist/spa/404.html` (SPA soft fallback)
   - `touch dist/spa/.nojekyll`
5. Upload `dist/spa` → `actions/deploy-pages`

Repo is the **user/org site** (`happy-tourist.github.io`) → always domain root (`publicPath: '/'`).

### Repo / Actions setup checklist

- GitHub Pages source: **GitHub Actions**.
- Repository variables: `VITE_COLYSEUS_URL`, `VITE_API_URL` (match prod WSS/HTTPS hosts).
- Permissions in workflow: `pages: write`, `id-token: write`.

## Files map

| Concern | Path |
|---------|------|
| Dev env | `.env.development` |
| Prod env defaults | `.env.production` |
| Types | `env.d.ts` |
| Router mode / publicPath | `quasar.config.ts` |
| Hash history factory | `src/router/index.ts` |
| Colyseus URL consumer | `src/boot/colyseus.ts` |
| CI / Pages | `.github/workflows/deploy.yml` |
| Deploy notes | `AGENTS.md` |

## Agent workflow

1. Confirm whether the task is local env, typed keys, router mode, or CI/Pages.
2. Edit only the files in the map above that the change requires.
3. Keep hash mode + domain-root `publicPath` unless the user explicitly changes hosting.
4. Run local verification from the client package root; fix failures before claiming done.

Typical commands (agent runs):

```bash
npm run lint
npm run typecheck
npx quasar build -m spa
```

Optional local smoke: `npx quasar dev` (loads `.env.development`).

## Anti-patterns

- Changing only `.env.production` and forgetting GitHub Actions `vars` (CI overrides at build).
- Adding `VITE_*` usage without updating `env.d.ts`.
- Setting `vueRouterMode: 'history'` for this Pages deploy.
- Using a project-pages `publicPath` like `/happy-tourist.github.io/` on a user/org site.
- Committing tokens or private server credentials into env files (these URLs are public client config only).
