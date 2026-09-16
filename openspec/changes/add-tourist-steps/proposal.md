## Why

Сейчас один легальный шаг сразу передаёт ход следующему: партия — «по клетке за раз», без ресурса и без заданий на коричневых тайлах. Нужна экономика **шагов** и **просмотров**, peek под тайлом (пока happy-path Correct/Wrong) и явный/авто конец хода — иначе нельзя строить ловушки, покупки и вопросы поверх поля. После первой реализации уточнены auto-end (не сжигать ход при доступном peek на `*`), соло (∞ только у просмотров + проигрыш по шагам) и дыры (непроходимы как цель). Дополнительно: leftover после leave/finish не должен оставаться с **0 шагов** на старте — переход хода в соло даёт **+1 шаг**.

## What Changes

- **BREAKING:** успешный move **больше не** передаёт ход; ход кончается кнопкой «Завершить ход», таймером 60 с, или когда нет доступных действий.
- У каждого seated — приватные счётчики **шагов** и **просмотров** (старт 0/0; при получении хода в мульти **+1 / +1**); 1 клетка = −1 шаг; peek = −1 просмотр (finite).
- На своём туриста на живом коричневом `*` — иконка глаза; модалка награды 1–3; кнопки «Правильно» / «Неправильно»; **тайл снимается только при «Правильно»**; при «Неправильно» / timeout / leave / endTurn с открытой модалкой — только −peek, тайл и награда остаются.
- **Несколько peek за ход** разрешены, пока peeks > 0 и есть турист на живом `*` (лимит «один peek за ход» снят).
- **Auto-end** только если нет легального хода **и** нет ситуации «peeks > 0 и свой unfinished на живом `*`»; иначе ход остаётся (кнопка / timeout).
- После Correct клетка становится **дырой**: фишка может **стоять** на ней, но **никто не может ходить на дыру** (подсказки цели не показывают дыру; сервер reject). Магазин / выкуп тайла — later.
- Рядом со своим аватаром — счётчики и кнопка «Завершить ход» (в соло кнопка не нужна).
- **Соло** (один eligible): просмотры → **∞**, шаги **конечные**; **без end-turn**; пока current не меняется — без повторных grant. Если ход **переходит** leftover’у (permanent leave бывшего current или финиш → единственный eligible) — **+1 шаг** (peek не начислять: уже ∞). Если соло наступило **без** смены current (ушёл/финишировал не-current) — только peeks∞ + 5:00, без нового шага. Дальше шаги с Correct; конец = **5:00 timer** или **steps = 0 и никто не на живом `*`** → lock time-expired, **разные** тексты модалок.
- После успешного хода **фокус остаётся** на ходившем туристе: при оставшихся шагах — красные клетки куда идти (не на дыры); при оставшихся просмотрах и живом `*` — глаз. Финиш на центре / finished — selection сбрасывается.
- Анимация «+N» в счётчики — около **2 с**.

## Scope

- **Пакеты:** client + server (room `tourist`); meta skills под контракт.
- **Capability ID:**
  - `game/move` — бюджеты; move не advance’ит; end-turn; auto-end по peeks+`*`; соло peeks∞ / finite steps / step-loss; become-current в соло → +1 шаг; timeout/leave/endTurn mid-peek = Wrong KEEP; keep-focus; дыры не landable;
  - `game/board` — преген наград; peek UX; Correct снимает тайл; Wrong KEEP; дыра stand-ok / land-forbid; глаз на живом `*`;
  - `game/presence` — счётчики (соло: ∞ только peeks); «Завершить ход»; соло-модалка; разные модалки timer vs steps-loss; +N ~2 с.
- **Экран:** Game (seated); spectators видят дыры, не видят чужие счётчики.
- **Контракт:** room `tourist`; messages move / peek / peekAnswer / end-turn (+ private budgets); sync `removedTaskKeys`.

## Out of scope

- Вопросы/ловушки на тайлах (только stub Correct/Wrong).
- **Магазин** / покупка тайлов обратно на поле / разблокировка застрявших (later).
- Покупка просмотров/ловушек за шаги.
- Смена геометрии layout / числа `*` / таймеров 60 с и 5 мин.
- Публичный показ чужих шагов/просмотров.
- Кап «добить до 10» в соло (отклонён).
- Старт партии **сразу в одного** (lobby alone) — продукт не стартует так; не специм отдельный solo-seed.
- Новые HTTP / auth / lobby.
- Ambient-подсветка всех peekable туристов/клеток.
- Принудительный авто-сдвиг с дыры после Correct (фишка может стоять).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `game/move`: экономика хода; end-turn / auto-end; соло peeks∞ + step-loss; become-current→соло +1 шаг; multi peek без лимита 1/ход; move без advance; keep-focus; дыры не landable.
- `game/board`: награды под `*`; peek; Correct → дыра stand-ok / land-forbid; Wrong KEEP; глаз при keep-focus на `*`.
- `game/presence`: счётчики; «Завершить ход»; соло-модалка (∞ peeks); разные тексты timer vs steps-loss; +N ~2 с.

## Impact

- **Server:** budgets (split infinite peeks vs finite steps в соло); auto-end; move reject на removed; solo step-loss → timeExpired; solo become-current +1 step; mocha на обновлённые SC.
- **Client:** targets без дыр; counters/модалки соло; i18n двух концовок.
- **Контракт:** согласованный client↔server без новых npm-пакетов.
- **Docs/skills:** meta + sibling AGENTS по `docs/projects-map.md`.

## References

- Explore (этот чат): D1 walkable→**unlandable** hole (stand ok); D2 carry +1/+1; D3 только `*`; auto-end = peeks+живой `*`; D5 дыра у всех; D8=B 28/14/6; соло peeks∞ / steps finite / loss steps0∧¬on `*`; timer+steps-loss разные тексты; без лимита 1 peek/ход; grant +1/+1 мульти; become-current→соло **+1 шаг** (leave/finish); без grant если current не менялся; lobby-alone out of scope; keep-focus; +N = 2 с; incorrect KEEP; магазин later.
- Main specs: `openspec/specs/game/{move,board,presence,pieces,start,finish}/spec.md`.
- Sibling AGENTS; `docs/projects-map.md`.
