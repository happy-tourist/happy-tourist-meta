## Why

Игрокам за столом нужны короткие приветствия и пожелания удачи без полноценного чата: фразы появляются комикс-облаками у presence-аватаров и видны всем в комнате, включая зрителей. Свободный текст в игре не планируется — только фиксированные пресеты.

## What Changes

- На Game у **своего** seated-маркера — affordance-облачко: клик → две кнопки пресетов («Всем привет», «Удачи»); после выбора пикер сразу закрывается.
- Выбор пресета уходит в room `tourist` и **рассылается всем** клиентам комнаты (seated и spectators); облако показывается у presence-маркера отправителя с учётом локальной раскладки слотов.
- Облако живёт **10 с**; одновременно у одного отправителя максимум **3** живых; после исчезновения одного можно снова; лимит зеркалится на сервере.
- Шлёт любой **seated + online** в любой момент (не только в свой ход); spectator и offline/grace — нет; свободный текст и произвольные строки **запрещены** (whitelist `presetId` на сервере).

## Scope

- **Capability ID:** `game/say` (**новый**).
- **Пакеты:** client (Game presence UX + room I/O через game store) и server (room `tourist` message accept/reject/broadcast).
- **Контракт:** room `tourist` — новое ephemeral message для пресетных реплик (не schema state); room `lobby` без изменений.

## Out of scope

- Свободный / произвольный текст, история чата, приватные сообщения.
- Quasar Notify / viewport toasts как носитель реплик.
- Отправка от spectators или от offline seat в reconnect grace.
- Привязка отправки к текущему ходу.
- Новые HTTP API, auth, theme, правила хода / board geometry.
- Persist реплик в schema или БД.

## Capabilities

### New Capabilities

- `game/say`: пресетные реплики у presence на Game — UI, room protocol, TTL/лимит, whitelist, видимость всем в комнате.

### Modified Capabilities

- (нет — `game/presence` остаётся про маркеры/grace; say — отдельный домен поверх presence layout.)

## Impact

- Server: handler входящего say-intent; whitelist + concurrent limit; broadcast всем в room `tourist`; mocha на accept/reject.
- Client: affordance + пикер + стек облаков у presence; listen broadcast; маппинг `presetId` → текст; лимит/TTL на UI в согласии с сервером.
- Meta: skills messages / game-board / rooms при необходимости после apply.

## References

- Explore (чат): D1–D3, Q1–Q5; A = любой seated+online в любой момент; capability `game/say`.
- Sibling AGENTS: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`.
- Related main specs: `openspec/specs/game/presence/spec.md` (layout слотов — без изменения требований).
