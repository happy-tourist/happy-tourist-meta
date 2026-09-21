## 1. Client — header brand logo + leave wiring



- [x] 1.1 Прочитать `design.md` (D1–D3, D5), `specs/ui/branding/spec.md` (SC-BRAND-01…05), `specs/game/leave/spec.md` (SC-LEAVE-01, SC-LEAVE-08), skills `.agents/skills/client/work-with-pages/SKILL.md`, `client-work-with-structure/SKILL.md`, `work-with-rooms/SKILL.md` и текущие `../happy-tourist.github.io/src/App.vue`, `AccountPage.vue`, `SupportPage.vue` — зафиксировать точки врезки

- [x] 1.2 Убедиться, что `../happy-tourist.github.io/src/assets/brand/logo.png` на месте; в shared header слева показать logo (~28–32px); убрать Material `logout` leave-кнопку (SC-BRAND-01, SC-LEAVE-01)

- [x] 1.3 Клик по logo: auth-экраны — decorative; lobby — noop; game — существующий `onExitClick`/confirm/`leaveGame`; остальные authenticated — navigate lobby; aria: на Game — `game.leave`, иначе home/lobby (SC-BRAND-02…04, SC-LEAVE-08)

- [x] 1.4 Удалить page-level кнопки «В лобби» (`auth.backToLobby`) с Account и Support; при мёртвом ключе i18n — убрать или reuse для aria (SC-BRAND-05)



## 2. Client — title, favicon, scaffold cleanup



- [x] 2.1 Прочитать `design.md` (D4), `specs/ui/branding/spec.md` (SC-BRAND-06…08) и текущие `../happy-tourist.github.io/package.json`, `index.html`, `public/`, `src/assets/quasar-logo-vertical.svg`, `src/pages/index/`

- [x] 2.2 `productName` → `Happy Tourist`; в `index.html` оставить только `favicon.ico` link; проверить, что `public/favicon.ico` — продуктовый (SC-BRAND-06, SC-BRAND-07)

- [x] 2.3 Удалить scaffold: `public/icons/favicon-*.png` (+ пустую `public/icons/`), `src/assets/quasar-logo-vertical.svg`, мёртвый `src/pages/index/(index).vue` (и пустые родительские index-папки при необходимости) (SC-BRAND-08)



## 3. Client — verify + meta skills



- [x] 3.1 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

- [x] 3.2 Обновить `.agents/skills/client/work-with-pages/SKILL.md` и `client-work-with-structure/SKILL.md` (logo left / leave via brand на Game; без page «В лобби»; title/favicon) по факту диффа; при необходимости строки в sibling client `AGENTS.md`



## 4. Client — stable clickable logo + size (follow-up)



- [x] 4.1 Прочитать обновлённые `design.md` (D2, D5), `specs/ui/branding/spec.md` (SC-BRAND-04, SC-BRAND-09, SC-BRAND-10) и текущий `../happy-tourist.github.io/src/App.vue`

- [x] 4.2 Всегда один interactive brand-control (убрать bare `img` / `v-if` decorative); auth → `lobby`; lobby noop; game leave; иначе → lobby (SC-BRAND-04, SC-BRAND-09)

- [x] 4.3 Высота logo ≥ 60px; включить актуальный прямоугольный `src/assets/brand/logo.png` (SC-BRAND-10)

- [x] 4.4 В `../happy-tourist.github.io`: `npm run lint` и `npm run typecheck`; при падении — починить

- [x] 4.5 Обновить skills/AGENTS (`work-with-pages`, `client-work-with-structure` и связанные) под always-button / auth→lobby / 60px

