# content/maps — delta: guest filter chrome + СНЯТО only for staff on list

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-69 | covered (ContentMaps — guest sees **no** filter chrome) |
| SC-MAP-70 | covered (ContentMaps — non-staff maps list hides soft-unpublished) |

Related: `content/packs` (same guest filter / СНЯТО list pattern); `ui/branding` (list page chrome). Soft-unpublish server rules — SC-MAP-21… (unchanged). `GET /api/content/maps` (`listMaps`) already omits soft-unpublished for non-staff; this delta adds **client** list omit as defense-in-depth — **no** server list-API / filter-query change. Map card capacity chrome (two rows) unchanged.

## MODIFIED Requirements

### Requirement: Maps list filters

The Maps list MUST support filters: **all** (default); **on moderation** (caller’s own open pending or needs_revision map requests); **drafts** (caller’s never-published maps without an open request); **mine** (maps where the caller is `createdBy`). There MUST NOT be a favorites filter or star for maps. Guests MUST only see in-catalog maps. For anonymous guests, the client MUST **hide the entire filter chrome** — including the **all** control — and MUST NOT leave identity filters visible as disabled controls.

#### Scenario [SC-MAP-33]: Filter on moderation shows own pending maps

- **GIVEN** creator A has map M pending and user B has map N pending
- **WHEN** A applies the on-moderation filter
- **THEN** A sees M and MUST NOT see N solely via that filter

#### Scenario [SC-MAP-34]: Filter mine shows own maps including drafts

- **GIVEN** creator A has in-catalog map M1 and never-published map M2
- **WHEN** A applies the mine filter
- **THEN** both M1 and M2 appear for A

#### Scenario [SC-MAP-69]: Guest sees only All filter on maps

- **GIVEN** an anonymous guest on the maps list
- **WHEN** the list page renders
- **THEN** no filter controls are shown (including **all** / «Все»)
- **AND** the filter row chrome itself is absent (not an empty button row)
- **AND** on-moderation, drafts, and mine filter controls are not shown
- **AND** only in-catalog maps are listed (effective filter remains **all**)

## ADDED Requirements

### Requirement: Maps list hides soft-unpublished for non-staff

On the Maps list, soft-unpublished maps (`hasLive` and not in catalog) MUST be shown **only to staff**. Non-staff users (including anonymous) MUST NOT see those maps as list cards. The server list already omits them for non-staff; the client MUST still omit any such row defensively if present in the payload. Staff MUST retain soft-unpublished chrome and unpublish/republish actions as already specified. Live deep-link reject for non-staff is unchanged (SC-MAP-21).

#### Scenario [SC-MAP-70]: Non-staff maps list omits soft-unpublished cards

- **GIVEN** soft-unpublished map M (`hasLive` and not in catalog) and a non-staff authenticated user U (including anonymous)
- **WHEN** U opens the Maps list
- **THEN** M MUST NOT appear as a list card for U
- **AND** staff MUST still see M with soft-unpublished chrome when applicable
