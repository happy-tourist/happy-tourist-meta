## Why

Навигация «в лобби» размазана: на Game — Material `logout`, на кабинете и поддержке — текстовые «В лобби», в табе браузера — scaffold-имя `happy-tourist-client` и дефолтные иконки Quasar. Нужен единый brand-якорь слева в шапке и продуктовый title/favicon. После первой реализации: смена decorative/button DOM давала скачок позиции; прямоугольный логотип при ~30px слишком мелкий.

## What Changes

- Слева в общей шапке — логотип Happy Tourist: ведёт в лобби (на Game — тот же consented leave + confirm, что сейчас у exit).
- Текстовые кнопки «В лобби» и отдельный exit-`logout` на Game убираются.
- Один и тот же интерактивный brand-control на всех экранах shared header (включая auth и Lobby), чтобы позиция не скакала; на auth клик → lobby (гость может отскочить auth-guard’ом); на Lobby клик — noop.
- Высота логотипа ≥ 60px; продуктовый прямоугольный `logo.png`.
- Таб браузера: title «Happy Tourist», favicon `.ico`; удаление scaffold PNG-favicon и неиспользуемого Quasar-логотипа.

## Scope

- **Capability ID:** `ui/branding` (новый), `game/leave` (modified)
- **Пакеты:** client (+ meta OpenSpec); server **без** изменений
- **UX:** общая шапка (Login / Lobby / Account / Support / Game и смежные); замена page-level «В лобби» на brand-control; стабильный chrome control; размер логотипа
- **Leave:** правила confirm / immediate leave без изменения семантики — меняется только визуальный control (logo вместо `logout`)
- **Browser chrome:** document title + favicon; cleanup scaffold brand assets
- **Язык:** accessible names RU через существующие locale keys где применимо

## Out of scope

- Отдельный dark-variant логотипа (один PNG ок на обеих темах)
- PNG-набор favicon разных размеров (только `.ico`)
- PWA / apple-touch / manifest icons
- Server, room protocol, gameplay
- Logout сессии в Lobby («Выйти») — остаётся
- «Назад в поддержку» / staff-nav — не «в лобби»

## Capabilities

### New Capabilities

- `ui/branding`: логотип в общей шапке как primary home/lobby control на всех экранах (включая auth → lobby); стабильный interactive control; высота ≥ 60px; document title «Happy Tourist»; favicon `.ico`; удаление устаревших scaffold brand assets

### Modified Capabilities

- `game/leave`: primary leave-to-lobby control на Game — brand logo (не Material `logout`); confirm/immediate rules без изменений; отдельный exit-only control больше не требуется

## Impact

- Client: shared header chrome; Account / Support page headers; browser title/favicon; удаление Quasar scaffold logo + PNG favicons; follow-up: единый control + размер/ассет
- Specs: новый `ui/branding`; delta `game/leave`
- Skills `work-with-pages` / structure — обновить при apply при расхождении с каноном шапки
- Server: нет

## References

- Explore 2026-09-21: header logo → lobby; Game = leave+confirm; title + ico; assets `logo.png` / `favicon.ico`; delete Quasar scaffold logos
- Explore follow-up 2026-09-21: always-clickable control (no layout jump); auth click → lobby; logo height ≥ 60px; rectangular product logo
- `docs/projects-map.md`; sibling `../happy-tourist.github.io/AGENTS.md`
- Main specs: `game/leave`, `ui/theme` (theme toggle справа без изменений поведения)
- Skills: `work-with-pages`, `client-work-with-structure`, `work-with-rooms` (consented leave)
