# content/maps — delta: hide identity filters for guests

Базовый канон: `openspec/specs/content/maps/spec.md`. Change: для anonymous identity-фильтры (moderation / drafts / mine) MUST быть скрыты в UI; гость видит in-catalog maps (filter **all**).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-MAP-69 | covered (client vitest — `ContentMaps` identity filters hidden) |

Related: `content/packs` (same hide pattern); guest create gates unchanged on server (`requireVerifiedEditor` → `registered_user_required`); list remains `GET /api/content/maps` + client-side filter chrome only (no server filter query params).

## MODIFIED Requirements

### Requirement: Maps list filters

The Maps list MUST support filters: **all** (default); **on moderation** (caller’s own open pending or needs_revision map requests); **drafts** (caller’s never-published maps without an open request); **mine** (maps where the caller is `createdBy`). There MUST NOT be a favorites filter or star for maps. Guests MUST only see in-catalog maps. Filters that require identity beyond catalog (moderation / drafts / mine) MUST be **hidden from the UI** for guests (MUST NOT remain visible as disabled controls).

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
- **WHEN** the filter chrome renders
- **THEN** the **all** filter control remains visible (identity filters use `v-if` / equivalent — MUST NOT remain as disabled controls)
- **AND** on-moderation, drafts, and mine filter controls are not shown
- **AND** only in-catalog maps are listed
