## 0. Server (out of scope)

- [x] 0.1 **Не трогать** `happy-tourist-server`: room messages (`peek` / `peekPlace` / `peekSubmit`), schema (`peekDifficulty` / `peekPlacements` / `answerCards` / …), resolve (+difficulty steps / keep bind), content HTTP/DB difficulty — всё as-is. Delta-сценарии SC-BOARD-43/44/47 в `specs/game/board` — baseline для chrome, не BE-работа. Verify: в diff apply нет правок sibling server.

## 1. Shared chrome + i18n (client)

- [x] 1.1 Перед работой: `design.md` Visual Spec + Decisions; `app.scss` `--pack-card-*` / `.peek-slot*`; `PackTaskSetCardTile` dots CSS; skills `work-with-styles` (pack-cards) и `work-with-localization`
- [x] 1.2 Добавить/обновить i18n для badge `Сложность: {word}` (лёгкая/средняя/сложная) для game peek и compose; убрать показ голой цифры как единственного difficulty copy в peek. **Не** переиспользовать `game.peekDifficulty` (`{n}`) и **не** `content.difficulty.{n}` (`1 — …`) как badge copy — отдельные word-only ключи (или эквивалент без цифры). Verify: ключи в `en-US` catalog; badge без `{n}`-only и без digit-prefix select labels
- [x] 1.3 Вынести/расширить CSS для peek + compose: empty dashed slot без label; filled + pool mini-card на `--pack-card-*`, radius 12, AR 156∶206; primary line + secondary description если есть; dots 6px/gap2, resting color = token текущего N (как `PackTaskSetCardTile` `data-diff`) — verify: light/dark. **Carve-out:** `PackTaskTile` сейчас шарит `.peek-slot-like` с compose — новые portrait/unlabeled стили только через peek/compose modifiers (или отдельный compact class на tile); **не** менять общий base так, чтобы list empty labels / compact sizes сломались (см. design Visual Spec)

## 2. Game peek modal (client)

- [x] 2.1 Перед работой: delta `specs/game/board/spec.md` SC-BOARD-50…53; текущий peek `q-dialog` на Game page; skill `work-with-game-board` / peek topic
- [x] 2.2 Реализовать шапку: 3 read-only dots (resting color = difficulty N) + word badge; question; empty unlabeled slots + filled mini-cards; pool mini-card grid ≤4×4 + scroll; modal width per Visual Spec (≈360–400 / ≤92vw); full-width primary submit (высота 28–36 / action-h 30) — verify: peeker/spectator ветки и place/clear/submit bindings сохраняются
- [x] 2.3 Vitest для SC-BOARD-50…53 / обновлённых 46/49 (header, empty unlabeled, filled mini-card, pool scroll contract, reuse без used-chrome, submit) — verify: `npm test` (релевантные peek/board тесты) в client

## 3. Task compose parity (client)

- [x] 3.1 Перед работой: delta `specs/content/packs/spec.md` SC-PACK-237/238/260/263/276/277; `PackTaskComposeDialog`; skill compose / pack-cards carve-out (`PackTaskTile` не переводить на peek AR)
- [x] 3.2 Заменить primary difficulty control на interactive dots (hover preview `1…K` в цвете K, click/tap commit K) + badge `Сложность: {word}` (sentence case, word-only i18n); слоты/пул mini-cards как в peek (вкл. secondary description если есть) — verify: difficulty остаётся 1|2|3 в модели задачи; `PackTaskTile` list chrome не переведён на peek AR
- [x] 3.3 Vitest SC-PACK-276/277 + обновлённые 237/238/260/263 — verify: `npm test` (task compose + PackTaskTile slot chrome где есть) в client; list tiles не обязаны совпадать с peek portrait

## 4. Quality gate (client)

- [x] 4.1 Перед работой: краткий проход `client-verify-code` / lint expectations из client AGENTS
- [x] 4.2 `npm run lint` и `npm run typecheck` в `happy-tourist.github.io` — verify: exit 0
