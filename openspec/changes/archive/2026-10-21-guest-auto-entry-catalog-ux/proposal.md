## Why

Неавторизованный пользователь по ссылке на лобби (и другие гостевые экраны) сейчас вынужден проходить Login и вручную выбирать гостя — лишнее трение для игры по приглашению. У гостя в каталогах наборов и карт по-прежнему видны бесполезные identity-фильтры, а на этих экранах из‑за крошек часто появляется лишний вертикальный скролл при коротком контенте.

## What Changes

- Без сессии на обычных защищённых экранах клиент автоматически оформляет anonymous-сессию и остаётся на целевом URL (лобби, игра, каталоги и т.п.).
- Deep-link на staff/admin без сессии ведёт сразу на Login (без auto-guest).
- В шапке у гостя вместо logout — иконка входа; Login доступен anonymous для upgrade; кнопка «Войти как гость» скрыта, если уже гость; после успешного входа/регистрации — возврат на исходный экран.
- У гостя на списках наборов и карт identity-фильтры скрыты (не disabled).
- Оболочка с breadcrumbs не даёт лишний page-scroll, когда контент влезает во viewport.

## Scope

- **Пакет:** client (`happy-tourist.github.io`). Server-контракт anonymous JWT / create pack·map для гостя без изменений.
- **Capability ID:**
  - `auth/login` — silent guest entry; Login для already-anonymous; hide guest control when already guest; post-auth return.
  - `ui/branding` — header control для гостя (иконка входа); layout без ложного скролла при breadcrumbs.
  - `content/packs` — скрытие identity-фильтров для anonymous.
  - `content/maps` — то же для фильтров карт.
- Гостевые ограничения create pack/map и play в лобби/игре — сохранить как уже принято (не расширять права гостя).

## Out of scope

- Новые OAuth/провайдеры, смена server auth API, email verification flows.
- Share-ссылки с query-параметрами комнат, clipboard invite UI.
- Staff moderation queue UX, create-pack verify modal redesign.
- Server-side filter endpoints (фильтры остаются client-side).
- Смена визуального стиля header-кнопок сверх замены logout → login для anonymous.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `auth/login`: auto anonymous на обычных `requiresAuth` без сессии; Login допускает already-anonymous; скрытие «Войти как гость» для already-guest; возврат после auth.
- `ui/branding`: у anonymous вместо session logout — иконка входа; page shell с breadcrumbs без ложного вертикального скролла при коротком контенте.
- `content/packs`: identity-фильтры для anonymous **скрыты**.
- `content/maps`: identity-фильтры для anonymous **скрыты**.

## Impact

- Client: router guards, Login, shared header, packs/maps list filter chrome, page/breadcrumb layout.
- Meta: sync client skills (`client-work-with-auth`, `work-with-pages` / shell, `work-with-stores/content`) under the new guard/header/filter rules.
- Server: без обязательных изменений — reuse existing anonymous JWT (`@colyseus/auth` `signInAnonymously` / `onRegisterAnonymously`), `tourist` `MyRoom.onAuth` JWT accept (lobby = listing-only `LobbyRoom`), catalog `GET` for guests via `auth.middleware()`, and create gates that reject `anonymous` (see `design.md` «Server contract»).
- Пользовательский путь: холодный заход на гостевые экраны = сразу гость; явный Login для регистрации/входа и для staff deep-link.

## References

- Explore-решения: silent guest на гостевых экранах; staff → Login; header login icon; hide filters packs+maps; hide guest button when already anonymous; scroll fix с breadcrumbs.
- `openspec/specs/auth/login/spec.md`, `ui/branding/spec.md`, `content/packs/spec.md`, `content/maps/spec.md`
- Sibling: `../happy-tourist.github.io/AGENTS.md`; client skills `client-work-with-auth`, `work-with-pages`, `work-with-lobby`, `work-with-styles`
- `docs/projects-map.md`
