## Why

Игрокам нужна возможность создавать собственные игровые карты (геометрия 10×10) и отправлять их на модерацию по тому же продуктовому пути, что и наборы карточек. Сейчас есть только зашитая туристическая раскладка и UGC-паки; карт как контента нет — нельзя копить каталог раскладок до будущего выбора карты+пака в комнате.

## What Changes

- UGC-карты: создание, paint-редактор, submit на модерацию, approve в публичный список.
- Список «Карты»: всем — одобренные; автору — ещё свои неопубликованные (в т.ч. до submit).
- Общая staff-очередь с паками (бейдж типа); тред автор↔staff; soft-unpublish; staff Edit после approve.
- Без личной коллекции карт; без привязки к tourist-room в этом change.

## Scope

- **Capability ID:** `content/maps` (новый; зеркало домена `content/packs`, без collection)
- **Пакеты:** client (`happy-tourist.github.io`) + server (`happy-tourist-server`) — HTTP JWT content API + UI «Карты» / редактор / модерация
- **Аудитория:** create/edit/submit — verified non-anonymous; просмотр списка approved — любой JWT (вкл. anonymous); staff — moderator|admin
- **Контракт:** только HTTP `/api/content/maps*` (+ расширение общей очереди/модерации); Colyseus room `tourist` не меняется

## Out of scope

- Выбор карты (и пака) при создании/входе в tourist-room; подмена `TOURIST_LAYOUT` в runtime
- Коллекция карт / add-to-collection
- Title/description карты (идентификация: мини-превью + автор + `игроки×туристы`)
- Валидация раскладки по сторонам / связности / фиксированный центр 2×2 (только min стартовых клеток)
- Block UI на client (как у паков — endpoint может остаться later)
- Замена peek-stub / wiring контента в игру

## Capabilities

### New Capabilities

- `content/maps`: UGC-карты 10×10, конфиг игроков/туристов 1–4, paint editor, submit/модерация/approve, публичный список без коллекции, soft-unpublish, staff edit после freeze автора

### Modified Capabilities

- (нет отдельных delta на `content/packs` — общая staff-очередь расширяется поведением maps; требования packs не меняются)

## Impact

- Server: таблицы/API карт, модерация и soft-unpublish по аналогии с packs; staff pending включает map-запросы
- Client: раздел «Карты» (lobby nav), create/editor, «На модерации», staff hub с типом map, мини-превью
- Игра / lobby create room: без изменений

## References

- Explore-решения: D1 out of game; D2 игроки×туристы 1–4, min стартов = произведение; D3draft автор видит unpublished на «Карты»; D4 список approved всем; D5 soft-unpublish; без коллекции; общая очередь; reject/staff edit как packs
- Аналог: `openspec/specs/content/packs/spec.md`
- Sibling: `happy-tourist-meta/docs/projects-map.md`; `../happy-tourist.github.io/AGENTS.md`; `../happy-tourist-server/AGENTS.md`
