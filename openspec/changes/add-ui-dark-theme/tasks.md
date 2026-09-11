## 1. Server — schema + theme HTTP

- [x] 1.1 Прочитать `design.md` (D5), `specs/ui/theme/spec.md` (SC-THEME-04, SC-THEME-05), skills `.agents/skills/server/work-with-database/SKILL.md`, `work-with-routes/SKILL.md`, `server-work-with-auth/SKILL.md` и текущие `../happy-tourist-server/src/db/schema.ts`, `src/app.config.ts` — зафиксировать колонку `theme` и путь POST
- [x] 1.2 Добавить nullable `theme` text в `colyseus_users` schema (без NOT NULL); обеспечить появление колонки на существующем SQLite `game.db` (ALTER / совместимый путь); verify: schema export содержит `theme`
- [x] 1.3 Добавить thin `POST` endpoint сохранения `{ theme: 'light' | 'dark' }` с JWT middleware: update по user id; reject без JWT / anonymous / invalid body (SC-THEME-04); verify: handler зарегистрирован в `createRouter`
- [x] 1.4 Добавить mocha-тесты с ID `SC-THEME-04` и `SC-THEME-05` (reject anonymous/unauth; registered save + theme в userdata после login); в `../happy-tourist-server`: `npm test`; при падении — починить

## 2. Client — Dark, header, persist

- [x] 2.1 Прочитать `design.md` (D1–D4, D6), `specs/ui/theme/spec.md` (SC-THEME-01…03, SC-THEME-06…07), skills `.agents/skills/client/work-with-styles/SKILL.md`, `client-work-with-auth/SKILL.md`, `colyseus-client/SKILL.md`, `work-with-stores/SKILL.md` и текущие `../happy-tourist.github.io/quasar.config.ts`, `App.vue`, `stores/auth.ts`
- [x] 2.2 Включить Quasar `Dark` plugin; boot/composable: unset → `auto`, иначе `light`/`dark`; guest → `localStorage`; registered → apply `user.theme` из userdata при `onChange`/login (SC-THEME-01, SC-THEME-03, SC-THEME-06); Colyseus HTTP save только из store/composable
- [x] 2.3 `App.vue`: общая `q-header` с toggle light↔dark на всех страницах (SC-THEME-02); после выбора — persist (guest local / registered POST); не менять board CSS в GamePage (SC-THEME-07)
- [x] 2.4 Пройти chrome Login/Lobby/Game: вторичный текст читаем в dark (замена/`body--dark`-friendly классы где нужно); доску не трогать
- [x] 2.5 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 3. Meta + validate

- [x] 3.1 Обновить `.agents/skills/client/work-with-styles/SKILL.md` (runtime Quasar Dark, header, guest local vs registered DB); при необходимости строки в server DB/routes skills и sibling AGENTS — по факту диффа
- [x] 3.2 Из корня meta: `openspec validate add-ui-dark-theme`; при готовности к merge — `/opsx-sync` / archive по отдельной просьбе
