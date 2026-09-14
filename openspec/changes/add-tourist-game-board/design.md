## Context

См. `proposal.md` (Why / Scope). Сейчас Game — шашечная 8×8 сетка с select/targets/`sendMove`; client constant `CHECKERS_ROOM = 'checkers'`; server `defineRoom` ключ `checkers` + tests/loadtest. Meta/skills/AGENTS и `openspec/config.yaml` context описывают онлайн-шашки. Пакеты: **client** (`../happy-tourist.github.io`), **server** (`../happy-tourist-server`, только имя room + согласованные тесты/loadtest), **meta** (docs/skills/AGENTS/openspec context).

## Goals / Non-Goals

**Goals:**

- Статичное поле tourist на Game (layout, цвета, adaptive, сплошной центр).
- Room `tourist` на client и server, чтобы lobby create/join/list работали при проверке.
- Убрать шашечный move UX с Game.
- Ребренд «шашки» → «Счастливый турист» / настольная игра в каноне агентов; skill `work-with-checkers` → `work-with-game`.

**Non-Goals:**

- Правила хода, flip заданий, фигурки, переработка schema/move logic сверх зачистки клиентского UX и rename room; отдельная dark-палитра тайлов.

## Decisions

### D1 — Рендер поля: CSS Grid, не SVG

- Сетка 10×10; пустые углы — без тайла (прозрачный слот / `visibility`).
- Центр: один элемент на `grid-area` 2×2 (сплошной жёлтый rounded rect), не четыре клетки.
- Остальные тайлы — неинтерактивные элементы (`div`), не `button`.
- Альтернатива SVG — отвергнута: оси-aligned rounded tiles проще и ближе к текущему Game page pattern.

### D2 — Layout как клиентская константа

- Карта типов клеток (`empty` | `start` | `task` | `center`) задаётся константой на client; server в этом change **не** синхронизирует геометрию поля.
- Альтернатива: schema board сразу — отложено (правила later).

### D3 — Размеры

- Сетка: `aspect-ratio: 1` на контейнере + `repeat(10, 1fr)` по осям (не `%` высоты — иначе rows = 0); `max-width: calc(10 * 60px + 9 * 6px)` → max tile 60px; `--gap: 6px`, `border-radius: 12px`.
- Контейнер поля: `width: 100%` в content area Game; на узком экране edge-to-edge относительно page content; на широком — естественный cap через max tile 60px.
- Цвета: start зелёный, task коричневый, center жёлтый — фиксированные fills, не зависят от Quasar Dark; фон дыр = фон страницы.

### D4 — Room rename `checkers` → `tourist`

- Client: `CHECKERS_ROOM` → `TOURIST_ROOM = 'tourist'` (или эквивалентное имя константы без «checkers»); все create/join/joinOrCreate/lobby filter/HTTP `/rooms/...`.
- Server: ключ регистрации в `app.config.ts` `tourist: defineRoom(MyRoom).enableRealtimeListing()`; `test/MyRoom.test.ts`, `package.json` loadtest `--room tourist`.
- Sibling AGENTS и skills: везде `tourist`.
- **BREAKING** для уже открытых клиентов на старом имени — приемлемо (ещё нет прод-аудитории шашек).

### D5 — Вырезать шашечный move UX на client

- Удалить с Game: selection, targets, `getTargets`, piece rendering, `sendMove` из UI; заголовки «белые/чёрные» / «ваш ход» упростить или нейтрализовать до появления правил.
- Store: оставить join/leave/lobby wiring; `sendMove`/board cell encoding шашек — убрать или оставить мёртвыми без UI (предпочтительно убрать/заглушить, чтобы не тащить шашки).

### D6 — Meta ребренд + `work-with-game`

- Переименовать папку skill `.agents/skills/server/work-with-checkers` → `work-with-game`; обновить frontmatter/body под «настольная игра / правила later», без русских шашек.
- Пройти meta AGENTS, docs, `openspec/config.yaml` context, client/server skills и sibling `AGENTS.md`: заменить шашки/checkers на «Счастливый турист» / board game / room `tourist`.
- Client skill `work-with-game-board`: переписать под статичное tourist-поле (без draughts CellValue/move hints как канон).

### D7 — Server runtime сверх rename

- Не реализовывать правила/board authority в этом change.
- Комментарии в `MyRoom` / schema stubs: убрать «checkers» формулировки при правке файлов ради rename/тестов.

## Risks / Trade-offs

- [Частичный rename пропущен] → lobby пустой / join fail — mitigation: один проход grep `checkers` по client+server+meta; прогон server tests + ручной create/join.
- [RENAMED+MODIFIED в delta specs] → archive путает заголовок requirement — mitigation: validate change; при sync проверить имена `Canonical tourist room name` / `Tourist board appearance…`.
- [Store ещё шлёт `move` / 8×8 board] → ложная готовность правил — mitigation: UI без вызовов; schema stubs без шашечной семантики в docs.

## Migration Plan

1. Meta: skills rename + prose (можно параллельно с кодом).
2. Server: rename room + tests/loadtest.
3. Client: room constant + статичное поле + вырез move UX.
4. Ручная проверка: login → lobby list → create/join → Game показывает поле.
5. Rollback: вернуть имя `checkers` и старый Game board (git revert change).

## Open Questions

- Нет (решения explore зафиксированы: D1–D8).
