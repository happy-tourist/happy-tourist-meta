## 1. Server — schema + theme HTTP

- [x] 1.1 Прочитать `design.md` (D5), `specs/ui/theme/spec.md` (SC-THEME-04, SC-THEME-05), skills `.agents/skills/server/work-with-database/SKILL.md`, `work-with-routes/SKILL.md`, `server-work-with-auth/SKILL.md` и текущие `../happy-tourist-server/src/db/schema.ts`, `src/app.config.ts` — зафиксировать колонку `theme` и путь POST
- [x] 1.2 Добавить nullable `theme` text в `colyseus_users` schema (без NOT NULL); обеспечить появление колонки на существующем SQLite `game.db` (ALTER / совместимый путь); verify: schema export содержит `theme`
- [x] 1.3 Добавить thin `POST` endpoint сохранения `{ theme: 'light' | 'dark' }` с JWT middleware: update по user id; reject без JWT / anonymous / invalid body (SC-THEME-04); verify: handler зарегистрирован в `createRouter`
- [x] 1.4 Добавить mocha-тесты с ID `SC-THEME-04` и `SC-THEME-05` (reject anonymous/unauth; registered save + theme в userdata после login); в `../happy-tourist-server`: `npm test`; при падении — починить

## 2. Client — Dark, header, persist

- [x] 2.1 Прочитать `design.md` (D1–D4, D6), `specs/ui/theme/spec.md` (SC-THEME-01…03, SC-THEME-06…07), skills `.agents/skills/client/work-with-styles/SKILL.md`, `client-work-with-auth/SKILL.md`, `colyseus-client/SKILL.md`, `work-with-stores/SKILL.md` и текущие `../happy-tourist.github.io/quasar.config.ts`, `App.vue`, `stores/auth.ts`
- [x] 2.2 Включить Quasar `Dark` plugin; boot/composable: unset → `auto`, иначе `light`/`dark`; guest → `localStorage`; registered → POST save + apply (SC-THEME-01, SC-THEME-03, SC-THEME-06); reload/restore via GET профиля — §5 (не JWT-only); HTTP только из store
- [x] 2.3 `App.vue`: общая `q-header` с toggle light↔dark на всех страницах (SC-THEME-02); после выбора — persist (guest local / registered POST); не менять board CSS в GamePage (SC-THEME-07)
- [x] 2.4 Пройти chrome Login/Lobby/Game: вторичный текст читаем в dark (замена/`body--dark`-friendly классы где нужно); доску не трогать
- [x] 2.5 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 3. Meta + validate (v1 theme)

- [x] 3.1 Обновить `.agents/skills/client/work-with-styles/SKILL.md` (runtime Quasar Dark, header, guest local vs registered DB); при необходимости строки в server DB/routes skills и sibling AGENTS — по факту диффа
- [x] 3.2 Из корня meta: `openspec validate add-ui-dark-theme`; при готовности к merge — `/opsx-sync` / archive по отдельной просьбе

## 4. Server — GET theme from profile (reload sync)

- [x] 4.1 Прочитать `design.md` (D8), `specs/ui/theme/spec.md` (SC-THEME-08, SC-THEME-09), skills `.agents/skills/server/work-with-routes/SKILL.md`, `server-work-with-auth/SKILL.md` и текущие `../happy-tourist-server/src/app.config.ts`, `test/theme.test.ts` — зафиксировать контракт GET
- [x] 4.2 Добавить thin `GET /api/theme` с JWT middleware: SELECT `theme` по user id; ответ `{ theme: 'light' | 'dark' | null }`; reject без JWT / anonymous (как POST); verify: handler в `createRouter`
- [x] 4.3 Добавить mocha-тесты с ID `SC-THEME-08` и `SC-THEME-09` (после POST GET отдаёт значение профиля при том же JWT без нового login); в `../happy-tourist-server`: `npm test`; при падении — починить

## 5. Client — restore via GET

- [x] 5.1 Прочитать `design.md` (D4), `specs/ui/theme/spec.md` (SC-THEME-08/09), skills `.agents/skills/client/work-with-styles/SKILL.md`, `work-with-stores/SKILL.md`, `colyseus-client/SKILL.md` и текущие `../happy-tourist.github.io/src/stores/theme.ts`, `App.vue`
- [x] 5.2 Registered session restore/`auth.ready`: читать тему через `client.http.get('/api/theme')` и apply; guest — только localStorage; не использовать JWT `user.theme` как единственный источник после reload (SC-THEME-08/09); HTTP только из store
- [x] 5.3 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

## 6. Meta — docs after reload sync

- [x] 6.1 Обновить skills routes/styles/stores (GET `/api/theme`, restore ≠ JWT-only) по факту диффа; при необходимости sibling AGENTS
- [x] 6.2 Из корня meta: `openspec validate add-ui-dark-theme`; sync/archive — по отдельной просьбе

## 7. Client — break restore GET storm (SC-THEME-10)

- [x] 7.1 Прочитать `design.md` (D9), `specs/ui/theme/spec.md` (SC-THEME-10) и текущие `../happy-tourist.github.io/src/App.vue`, `src/stores/theme.ts` — зафиксировать петлю watch → GET → patch `auth.user`
- [x] 7.2 Стабилизировать trigger restore: `watch` по примитивам/массиву источников (не getter с новым массивом каждый раз); после GET не заменять `auth.user` ради `theme` (тема в theme store); POST toggle может патчить userdata без повторного GET (SC-THEME-10)
- [x] 7.3 Вручную/DevTools: registered session ready → один (или конечное малое число) `GET /api/theme`, без пачки после apply; guest без GET; в `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить
- [x] 7.4 Обновить client skills stores/styles (и при необходимости align) по факту фикса петли; из корня meta: `openspec validate add-ui-dark-theme`
