## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-06 | revised **card chrome** (client vitest — preview + capacity rows, no author on card); **server mocha** unchanged for list visibility + list JSON (`authorDisplayName` / seats / grid still returned) |
| SC-MAP-16 | revised **card chrome** (client vitest — approve → list card without author); **server mocha** unchanged for approve → in-catalog + list visibility |
| SC-MAP-31 | revised **badge chrome** (client vitest — short «НА ПРОВЕРКЕ»; detail via SC-MAP-66/67) |
| SC-MAP-32 | revised **badge chrome** (client vitest — short «ДОРАБОТАТЬ»; detail via SC-MAP-66/67) |
| SC-MAP-45 | covered / reinforced (no published / in_catalog badge; no «ОПУБЛИКОВАНО») |
| SC-MAP-47 | unchanged (View still shows author + seats from API) |
| SC-MAP-55 | revised (client vitest — ~180 card; two capacity rows; outline actions; no author) |
| SC-MAP-66 | covered (client vitest — centered overlay status; short badge copy) |
| SC-MAP-67 | covered (client vitest — pack-card chrome tokens; muted dashed; outline+icon actions) |

Related: pack/task-set chrome SC-PACK-239…248 / 255 (shared `--pack-card-*`; map cards now consume). View author meta SC-MAP-46/47 unchanged. **No new server/HTTP claims** — list/unpublish/republish contracts stay as in main `content/maps` + sibling `listMaps`.

## MODIFIED Requirements

### Requirement: Maps list without collection

The client MUST expose a **Maps** section (reachable from the lobby) showing a shared list of **approved in-catalog** maps to any authenticated user (including anonymous). Each list **card** MUST show a **mini preview** of the grid and the seat config as **two labeled rows** (players count and tourists-per-player count) — NOT a single `players×tourists` string as the only capacity chrome, and MUST **NOT** show the author display identity on the card. The same Maps page MUST additionally list the **creator’s own unpublished** maps (never approved, or soft-unpublished only for staff visibility rules below) so the author can reopen the editor before and without a moderation submit. Non-authors MUST NOT see another user’s unpublished maps on that list. There MUST be a **Create map** affordance at the top of the Maps section for eligible users. The system MUST NOT offer add-to-collection or remove-from-collection for maps. Author identity MUST remain available on the map **View** surface (SC-MAP-47) and staff queue preview; this requirement only removes author from the list card.

#### Scenario [SC-MAP-06]: Approved maps visible to all sessions

- **GIVEN** an approved in-catalog map M and any authenticated user U (including anonymous)
- **WHEN** U opens the Maps list
- **THEN** M appears as a card with mini preview and players / tourists capacity rows
- **AND** the card MUST NOT show the author display name

#### Scenario [SC-MAP-07]: Author sees own unpublished on Maps

- **GIVEN** creator A has unpublished map M that was never approved
- **WHEN** A opens the Maps list
- **THEN** M appears for A and is enterable for edit
- **AND** another non-staff user B does not see M on the Maps list

#### Scenario [SC-MAP-08]: No collection for maps

- **GIVEN** an approved map M
- **WHEN** any user views Maps or the map detail
- **THEN** no collect / uncollect affordance is offered

### Requirement: Approve publishes into Maps list and freezes author edit

When staff approves a map moderation request (after take), the submitted grid and seat config MUST become the live snapshot, the map MUST become **in catalog**, and it MUST appear on the public Maps list. After the map has been approved at least once, non-staff users **other than the creator** MUST NOT edit the live map. The **creator** MUST be able to Edit under the exclusive lock and submit further changes through moderation (see author re-edit). **Staff** (moderator|admin) MUST be able to Edit any map under exclusive lock and save directly to live/working **when** no open author map request blocks staff content edit and the lock is free. Never-approved maps remain editable by the creator (and staff).

#### Scenario [SC-MAP-16]: Approve lists map publicly

- **GIVEN** a pending map request and staff who has taken the request
- **WHEN** the actor approves
- **THEN** the map is in catalog
- **AND** any authenticated user sees it on the Maps list with mini preview and players / tourists capacity rows
- **AND** the list card MUST NOT show the author display name

#### Scenario [SC-MAP-17]: Author cannot edit after approve

- **GIVEN** map M approved into catalog and creator A (non-staff)
- **WHEN** A attempts a **direct live** edit/save without going through moderation submit
- **THEN** the system rejects applying those changes to live without moderation
- **AND** A MAY still acquire the edit lock, amend a working copy, and submit for moderation (see author re-edit)

#### Scenario [SC-MAP-18]: Staff edits under exclusive lock

- **GIVEN** approved map M with no open author request and free edit lock, and staff S
- **WHEN** S acquires the edit lock and saves grid or seat config changes
- **THEN** the live map reflects those changes without a new author moderation submit
- **AND** a second staff actor cannot acquire the lock while S holds it

### Requirement: Maps list shows moderation statuses

The Maps list MUST show status badges for **pending**, **needs_revision**, **draft** (never-published without open request or author draft after cancel), and soft-**unpublished** (staff) when applicable. Rows that are simply published / in catalog MUST **NOT** show an «В каталоге» / in_catalog status badge and MUST **NOT** show an «ОПУБЛИКОВАНО» / published badge. A map with an open request MUST show pending or needs_revision rather than only a generic draft badge. Non-authors still MUST NOT see another user’s never-published maps. Badge **copy** on the card MUST use the same short uppercase product sense as pack/task-set cards («ЧЕРНОВИК», «НА ПРОВЕРКЕ», «ДОРАБОТАТЬ», «СНЯТО») — not long list strings such as «Нужна доработка» or «Снято с публикации» on the pill. Badge placement: **horizontally centered overlay** on the mini preview (not a body strip below the preview). Soft-unpublished cards MUST use muted list chrome (reduced opacity and dashed border) consistent with soft-unpublished pack cards.

#### Scenario [SC-MAP-31]: Pending map shows pending on Maps list

- **GIVEN** creator A submitted map M and the request is pending
- **WHEN** A opens the Maps list
- **THEN** M appears with a pending status indication using short product sense «НА ПРОВЕРКЕ» (or equivalent i18n)

#### Scenario [SC-MAP-32]: Needs-revision map shows needs-revision on Maps list

- **GIVEN** creator A has map M with an open needs_revision request
- **WHEN** A opens the Maps list
- **THEN** M appears with a needs_revision status indication using short product sense «ДОРАБОТАТЬ» (red outline product sense; not long «Нужна доработка»)

#### Scenario [SC-MAP-45]: Published map has no in_catalog badge

- **GIVEN** an approved in-catalog map M with no open author-facing draft/pending/needs_revision mark for the caller
- **WHEN** the user views M on the Maps list
- **THEN** M has no «В каталоге» / in_catalog status badge
- **AND** M has no «ОПУБЛИКОВАНО» / published status badge

### Requirement: Maps list uses card tiles with mini preview

The Maps section list MUST render each map as a card in a wrapping row with resting width about **180** CSS pixels (height MAY grow with preview, capacity rows, and stacked actions). Each card MUST show a **mini preview** of the grid in the upper area and, below that preview, **two** capacity rows: a players row (leading person-style icon, uppercase label product sense «ИГРОКОВ:», count right) and a tourists-per-player row (leading backpack-style icon, uppercase label product sense «ТУРИСТОВ:», count right), with a pale horizontal divider between those rows. Author identity MUST **NOT** appear on the card. Status chrome, when applicable, MUST overlay the preview centered horizontally. Action controls (Edit, soft-unpublish, republish, and similar) MUST appear at the **bottom** as stacked full-width **outline** buttons with a leading icon and text label (slim height ~28–32 CSS px), using short soft-unpublish / republish labels product sense «Снять» / «Вернуть» on the card (confirm dialogs MAY keep long maps unpublish copy). Soft-unpublished cards MUST use muted chrome (opacity about **0.72** and dashed border). Resting surface, hover (border + soft shadow only, **no** scale), splitter, and action outline MUST match the shared pack-card chrome product sense already used by pack catalog and task-set cards. Existing open/navigation and visibility rules for unpublished maps MUST remain.

#### Scenario [SC-MAP-55]: Maps list renders cards with mini preview and capacity

- **GIVEN** an authenticated user on the Maps section with at least one listed map
- **WHEN** the user views the list
- **THEN** each map is shown as a rounded card about 180 CSS pixels wide
- **AND** the card shows a mini grid preview above two capacity rows (players and tourists), not a single `players×tourists`-only line as the sole capacity chrome
- **AND** the card MUST NOT show the author display name
- **AND** Edit when available is a bottom full-width outline+icon control

## ADDED Requirements

### Requirement: Map list card status overlay and short badges

When a Maps list card shows a moderation/status badge, the badge MUST be drawn as an overlay on the mini preview, **horizontally centered** near the top of the preview. Soft muted pills (draft / unpublished / pending amber) and revise red-outline MUST follow the same product sense as pack/task-set status badges. Badge icons for draft / pending / unpublished MAY reuse the same document / clock / eye-off product family as task-set badges; revise MAY be text-only red outline (catalog pack sense). Confirm / header long copy for soft-unpublish MUST NOT replace the short card badge «СНЯТО».

#### Scenario [SC-MAP-66]: Status badge centered on preview with short copy

- **GIVEN** a Maps list card whose author-facing status is draft, pending, needs_revision, or soft-unpublished
- **WHEN** the user views the card
- **THEN** the status badge overlays the mini preview and is horizontally centered
- **AND** the badge uses the short uppercase product label for that status («ЧЕРНОВИК» / «НА ПРОВЕРКЕ» / «ДОРАБОТАТЬ» / «СНЯТО»)
- **AND** a clean published in-catalog card shows no status badge on the preview

### Requirement: Map list cards share pack-card chrome and outline actions

Maps list cards MUST consume the shared pack-card resting chrome tokens (background, foreground, muted ink, border, hover border/shadow, splitter, action height) so they match pack catalog and task-set cards in product sense. Card soft-unpublish muted MUST use opacity about **0.72** and a **dashed** border. Bottom actions MUST be outline+icon (Edit with pencil-style icon; soft-unpublish with eye-off; republish with eye) with short «Снять» / «Вернуть» labels on the card. Hover MUST change border and soft shadow only (**no** enlarge/scale). Lead icons for players and tourists MUST be present (custom SVG preferred; Material placeholder allowed until assets land).

#### Scenario [SC-MAP-67]: Map cards match pack chrome; muted dashed; short outline actions

- **GIVEN** the Maps list rendered in the same theme as the packs catalog
- **WHEN** the user compares a resting map card to a resting pack catalog card
- **THEN** both use the shared pack-card surface tokens for background, border, and outline action border product sense
- **AND WHEN** a soft-unpublished map card is shown
- **THEN** that card uses opacity about 0.72 and a dashed border
- **AND WHEN** staff soft-unpublish / republish actions apply
- **THEN** the card shows short «Снять» / «Вернуть» outline+icon controls (not long «Снять с публикации» on the card button)
