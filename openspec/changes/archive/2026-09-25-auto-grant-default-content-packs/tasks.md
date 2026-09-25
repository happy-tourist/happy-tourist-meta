## 1. Server — helpers + env

- [x] 1.1 Прочитать `design.md`, delta `specs/content/packs/spec.md` (SC-PACK-142…147), skills `server-work-with-structure` / `work-with-config` / `server-work-with-auth`; сверить `addToCollection` / `backfillLegacyEmailVerified` как аналоги
- [x] 1.2 Добавить parse CSV + eligible filter + `grantDefaultPacksToUser` + per-pack boot backfill (`db.configs` key) по design D1–D5; пустой env = no-op; неопубликованные id skip без throw — unit/mocha на parse/skip при возможности
- [x] 1.3 Документировать `DEFAULT_CONTENT_PACK_IDS` в `.env.example` и таблице env server `AGENTS.md` рядом с `BOOTSTRAP_ADMIN_IDS`

## 2. Server — boot + create-user hooks

- [x] 2.1 Вызвать backfill на старте в `express()` после ensure content tables (рядом с bootstrap admins); повторный старт идемпотентен — mocha SC-PACK-142/146/147
- [x] 2.2 Врезать grant при создании user: email register, Google OAuth, anonymous (`signInAnonymously`); **не** вызывать на login / `onParseToken` / `GET /collection` — mocha SC-PACK-143
- [x] 2.3 После remove default-пака повторный login + `GET /collection` не возвращают пак; ручной `POST /collection` по-прежнему работает — mocha SC-PACK-145
- [x] 2.4 Неeligible id (missing / no live / soft-unpublish / blocked) не попадают в коллекции — mocha SC-PACK-144

## 3. Server — verify

- [x] 3.1 `npm test` (mocha) с покрытием SC-PACK-142…147; при падении — починить
- [x] 3.2 `npm run build` в `happy-tourist-server` успешен
