## Why

В шапке до сих пор один маскот-логотип на обе темы, а рядом с кнопками входа, выхода из сессии и выхода из игры показывается лишний текст — chrome выглядит шумнее, чем нужно. Готовы theme-aware wordmark-ассеты: пора заменить mark и убрать видимые подписи у этих кнопок везде в клиенте.

## What Changes

- Shared header показывает wordmark Happy Tourist, зависящий от светлой/тёмной темы (обрезанный по контенту ассет).
- Старый маскот-ассет header-логотипа удаляется из клиентского пакета.
- Кнопки входа, выхода из сессии и выхода из игры — только иконка (+ accessible name), без видимого текста; то же для кнопки выхода в кабинете.
- Высота wordmark почти на всю высоту области шапки (небольшой вертикальный зазор).

## Scope

- **Capability ID:** `ui/branding` (theme-aware brand mark; icon-only session login/logout в shared header и на Account; удаление старого header logo asset).
- **Capability ID:** `game/leave` (правый leave-control на Game без видимого label — только иконка + accessible name).
- **Пакет:** client only (`happy-tourist.github.io`).
- **Server scope:** нет (контракт client↔server не меняется).
- Поверхности: shared header на всех экранах с chrome; Account (кнопка выхода из сессии).

## Out of scope

- Server / Colyseus / HTTP / schema / room leave protocol (consented leave / `onLeave` as-is).
- Смена favicon / document title.
- Burger-меню: пункты входа/выхода остаются текстовыми (это меню, не toolbar-подпись).
- Новые i18n-строки сверх уже существующих accessible names.
- Редизайн остальных header-контролов (Packs/Maps/Support, theme, account).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `ui/branding`: theme-aware wordmark вместо единого маскота; tight crop ассетов; высота mark ≈ высота content area шапки; icon-only session login/logout в toolbar; icon-only logout на Account; удаление старого `logo.png`-маскота из пакета.
- `game/leave`: правый leave-control без видимого текста (icon-only + accessible name); поведение leave/confirm не меняется.

## Impact

- Client UI chrome и brand assets; unit-тесты header chrome / leave.
- Meta: синхронизация client skills / AGENTS / `docs/projects-map.md` `client.assets` (logo paths, 28px, icon-only) — см. `tasks.md` §5.
- Контракт client↔server не меняется.
- Breaking для визуала: header mark и подписи у auth/leave кнопок.

## References

- Main specs: `openspec/specs/ui/branding/spec.md`, `openspec/specs/game/leave/spec.md`
- Sibling: `happy-tourist.github.io/AGENTS.md`
- Карта путей: `docs/projects-map.md`
