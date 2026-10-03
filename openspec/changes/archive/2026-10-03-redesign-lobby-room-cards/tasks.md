## 1. Server — listing metadata taskCount

- [x] 1.1 В `refreshMetadata` (tourist room) добавить `taskCount` на каждый элемент `taskSetLabels` = число tasks snapshot с тем же `taskSetId`; сохранить совместимые поля — verify: mocha SC-LOBBY-32 (create → lobby listing metadata counts) в `happy-tourist-server` (`npm test`, фильтр lobby/rooms metadata)
- [x] 1.2 Обновить/добавить server тест на listing metadata после create с ≥2 sets — verify: SC-LOBBY-32 зелёный; старые поля seats/maxSeats/packTitle/mapGrid не регрессируют

## 2. Client — tile + Lobby wiring

- [x] 2.1 Добавить lobby room card tile (~180, preview, seats, tourists, pack title, set rows ≤4 + overflow, status overlay slot, outline Войти); alias `--pack-card-*`; lead masks `url("${…}")` — verify: компонент рендерится в isolation / story-less vitest mount
- [x] 2.2 Заменить list-row listing на `.pack-card-grid` + tile в Lobby; join по карточке и кнопке; busy-lock SC-LOBBY-19 — verify: ручной/vitest join не двойной; testids seats/sets/join
- [x] 2.3 i18n: tourists без «+», short ОЖИДАНИЕ/ИГРА, short «Набор #{n}» (если нужен отдельный ключ), overflow «ещё {k}» — verify: строки видны на карточке (RU catalog)
- [x] 2.4 Типы `GameRoomMeta.taskSetLabels` + `taskCount`; иконка seats — Material `event_seat` или mask `lobby-room-card-seats.svg` если файл уже есть; tourists/set — reuse существующих SVG — verify: нет broken mask square (quoted url)

## 3. Tests + skills

- [x] 3.1 Client vitest: SC-LOBBY-25/26/33/34/35 (preview, seats/tourists, pack+counts, chrome 180+outline, status overlay, overflow) — verify: `npm test` в client, релевантные Lobby* тесты зелёные
- [x] 3.2 Обновить skills `work-with-lobby` (+ note в `pack-cards.md` что lobby listing потребляет `--pack-card-*`) — verify: топики упоминают card listing / taskCount metadata
- [x] 3.3 Скопировать mock в `openspec/changes/redesign-lobby-room-cards/assets/` (если файл доступен) — verify: путь ссылается из design Visual Spec

## 4. Seats/tourists metric rows (explore follow-up)

- [x] 4.1 `LobbyRoomCardTile`: seats/tourists — full-width grid `lead | label | count` как set-rows (убрать centered `max-content`); иконки на одной вертикали с наборами — verify: DOM/CSS grid columns совпадают с set-row; testids label+count
- [x] 4.2 i18n: склоняемые labels мест по `maxSeats` (`Место`/`Места`/`Мест`) и туристов по `touristsPerPlayer` (`Турист`/`Туриста`/`Туристов`); справа `capacity` / bare `n` без «+» — verify: RU catalog + tile render (1/2/5 формы)
- [x] 4.3 Vitest SC-LOBBY-25/36 (+ обновить устаревшие asserts про combined «N ТУРИСТ…» / centering) — verify: `npm test` LobbyRoomCardTile (+ LobbyCreateWire если ломается)
- [x] 4.4 Skills note (`work-with-lobby` / listing-cards): metric rows = set-row rhythm; plural by maxSeats / touristsPerPlayer — verify: топик описывает layout + copy
