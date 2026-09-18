---
name: work-with-env-deploy
description: >-
  Use when changing server env vars, PM2, VPS deploy, GitHub Actions rsync, or
  DATABASE_URL for happy-tourist-server — AUTH_SALT / JWT_SECRET / SESSION_SECRET /
  GOOGLE_CLIENT_ID / GOOGLE_CLIENT_SECRET, SMTP_BZ_* / MAIL_FROM /
  AUTH_BACKEND_URL / CLIENT_APP_URL, .env.development / .env.production,
  ecosystem.config.cjs, or .github/workflows/deploy.yml.
---

# Work With Env And Deploy

Use this skill for **env vars**, **PM2**, and **VPS deploy via GitHub Actions** of the Colyseus tourist server (`happy-tourist-server`).

Skills path for now: `.agents/skills/server/` in this repo (canonical copy may later live under `happy-tourist-meta`). Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io` (deploys to GitHub Pages separately).

Deploy target: VPS under `/var/www/happy-tourist-server`, Node 22, PM2. Trigger: push to `main` (or `workflow_dispatch`).

## Hard rules

| Do | Don't |
|----|--------|
| Keep production secrets only on the server (`.env.production`) | Commit prod secrets or `.env.production` with real values |
| Keep rsync excludes for `.env*` and `game.db*` | Let CI overwrite server DB or prod env |
| Document new env keys in `.env.example` | Invent CI-injected app secrets (unlike the client; app secrets stay on VPS) |
| Use GH secrets `SSH_*` only for deploy SSH | Put `AUTH_SALT` / `JWT_SECRET` / `SESSION_SECRET` / `GOOGLE_CLIENT_*` / `SMTP_BZ_*` in GitHub Actions vars |
| Run local `npm test` / `build` / `dev` from server package root | Skip verification or assume pass without running |

## Env vars

`@colyseus/tools` loads `.env.${NODE_ENV}` if present, else `.env`.

| Variable | Role | Local | Prod (server file) |
|----------|------|-------|--------------------|
| `AUTH_SALT` | `@colyseus/auth` password salt | `.env.development` | `/var/www/.../.env.production` |
| `JWT_SECRET` | JWT sign/verify (rooms `onAuth`) | same | same |
| `SESSION_SECRET` | Auth session | same | same |
| `GOOGLE_CLIENT_ID` | Google OAuth Web client ID | same | same |
| `GOOGLE_CLIENT_SECRET` | Google OAuth Web client secret | same | same |
| `SMTP_BZ_HOST` | smtp.bz SMTP host (`src/lib/mailer.ts`) | same | same |
| `SMTP_BZ_PORT` | SMTP port (default `587`) | same | same |
| `SMTP_BZ_USER` / `SMTP_BZ_PASS` | smtp.bz credentials | same | same |
| `MAIL_FROM` | From header (e.g. `Happy Tourist <noreply@happy-tourist.ru>`) | same | same |
| `AUTH_BACKEND_URL` | Public API origin for confirm/reset links (`auth.backend_url`) | e.g. `http://localhost:2567` | `https://api.happy-tourist.ru` |
| `CLIENT_APP_URL` | Client origin; confirm success → `{CLIENT_APP_URL}/#/lobby` | e.g. `http://localhost:9000` | `https://happy-tourist.github.io` |
| `DATABASE_URL` | SQLite path for GameDatabase | `./game.db` | often `/var/www/happy-tourist-server/game.db` |
| `NODE_ENV` | `development` / `production` (CORS, monitor/playground) | `development` | `production` (also set in PM2 `env`) |
| `PORT` | Listen port | `2567` | `2567` (PM2 `env` + file) |

Generate secrets: `openssl rand -base64 32`. Template: `.env.example`.

Google OAuth also needs Authorized redirect URI in Google Cloud Console (`http://localhost:2567/auth/provider/google/callback` locally; `https://<api-host>/auth/provider/google/callback` in prod) — documented in `.env.example`, not injected by CI.

Mail delivery (smtp.bz) also needs DNS/SPF(+DKIM) for `happy-tourist.ru` on the ops side — outside CI.

When adding a new env key:

1. Add to `.env.example` with a comment.
2. Add to local `.env.development` (and document prod placement on the VPS).
3. Do **not** rely on GitHub Actions to inject app secrets — update the server `.env.production` manually (or via secure SSH), then `pm2 reload ... --update-env` if needed.
4. If PM2 must expose it, add to `ecosystem.config.cjs` `env` only when it is non-secret runtime config (prefer the `.env.production` file for secrets).

## PM2 (`ecosystem.config.cjs`)

| Setting | Value |
|---------|-------|
| `name` | `happy-tourist-server` |
| `script` | `build/index.js` |
| `cwd` | `/var/www/happy-tourist-server` |
| `instances` / `exec_mode` | `1` / `fork` |
| `wait_ready` | `true` (`listen_timeout` 10000, `kill_timeout` 5000) |
| `max_memory_restart` | `500M` |
| `node_args` / `NODE_OPTIONS` | `--max-old-space-size=350` |
| `env.PORT` | `2567` |
| `env.NODE_ENV` | `production` |
| Logs | `/var/log/happy-tourist-server/error.log`, `out.log` |

Sized for ~1 GB VPS. Do not switch to cluster without revisiting memory limits.

## GitHub Actions deploy

Workflow: `.github/workflows/deploy.yml` (`Deploy happy-tourist-server`).

Triggers: `push` to `main`, `workflow_dispatch`.

### Pipeline (must stay in sync)

1. `actions/setup-node` **Node 22** + npm cache.
2. CI compile check: `npm ci` && `npm run build` (artifact not shipped).
3. SSH from secrets: `SSH_PRIVATE_KEY`, `SSH_KNOWN_HOSTS`, `SSH_PORT`, `SSH_USER`, `SSH_HOST`.
4. `rsync -az --delete` to `/var/www/happy-tourist-server/`, **excluding**:
   - `.git`
   - `node_modules`
   - `build`
   - `.env` / `.env.*`
   - `game.db*`
5. Remote: `cd /var/www/happy-tourist-server` → `npm ci` → `npm run build` → `pm2 reload ecosystem.config.cjs --update-env` (or `pm2 start` if first run) → `pm2 save`.

Sibling client Pages deploy is a separate repo/workflow — do not mix client `VITE_*` into this pipeline.

### Repo / Actions / VPS checklist

- GitHub secrets: `SSH_PRIVATE_KEY`, `SSH_KNOWN_HOSTS`, `SSH_PORT`, `SSH_USER`, `SSH_HOST`.
- On VPS once: Node 22, PM2, log dir `/var/log/happy-tourist-server/`, `.env.production` with real secrets + `DATABASE_URL` + mail (`SMTP_BZ_*`, `MAIL_FROM`) + `AUTH_BACKEND_URL` / `CLIENT_APP_URL`, writable SQLite path.
- Confirm rsync never ships `.env*` or `game.db*`.

## Files map

| Concern | Path |
|---------|------|
| Env template | `.env.example` |
| Local env | `.env.development` |
| Prod env (server only) | `.env.production` on VPS — not committed with secrets |
| PM2 | `ecosystem.config.cjs` |
| CI / rsync / remote build | `.github/workflows/deploy.yml` |
| DB wiring | `src/db/index.ts`, `src/db/schema.ts` |
| CORS / health (prod vs dev) | `src/app.config.ts` |
| Deploy notes | `AGENTS.md` |

## Agent workflow

1. Confirm whether the task is local env, prod secrets on VPS, `DATABASE_URL`, PM2 memory/port, or CI/rsync.
2. Edit only the files in the map above that the change requires.
3. Keep rsync excludes and “secrets on server only” unless the user explicitly changes the deploy model.
4. Run local npm verification from the server package root; fix failures before claiming done. Remote prod deploy / VPS smoke (SSH credentials) — propose steps to the user when the agent cannot execute them.

Typical commands (agent runs locally):

```bash
npm run build
npm test
npm run dev
```

Prod smoke (VPS / prod host — propose when agent lacks SSH access):

```bash
curl -sS http://127.0.0.1:2567/health
pm2 status
pm2 logs happy-tourist-server --lines 50
```

## Anti-patterns

- Committing `.env.production` with real `AUTH_SALT` / `JWT_SECRET` / `SESSION_SECRET` / `GOOGLE_CLIENT_*` / `SMTP_BZ_*`.
- Removing `.env*` or `game.db*` from rsync excludes (wipes prod secrets/DB on deploy).
- Putting app secrets in GitHub Actions `vars`/`secrets` and expecting the Node process to see them without a server-side `.env.production`.
- Raising PM2/Node memory past ~350–500M on a 1 GB VPS without a plan.
- Assuming client GitHub Pages deploy updates this server (separate pipeline).
- Changing `DATABASE_URL` without ensuring the SQLite file path exists and is writable on the VPS.
- Leaving `AUTH_BACKEND_URL` / `CLIENT_APP_URL` wrong in prod (broken confirm/reset links or post-confirm redirect).
