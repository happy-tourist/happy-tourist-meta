# content/packs — delta: hide identity filters for guests

Базовый канон: `openspec/specs/content/packs/spec.md`. Change: для anonymous identity-фильтры (moderation / drafts / mine / favorites) MUST быть скрыты в UI, не лишь disabled; гость по-прежнему видит публичный каталог (filter **all**).

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-PACK-153 | covered (client vitest — `ContentUnifiedList` identity filters hidden) |
| SC-PACK-286 | covered (client vitest — guest sees only All filter) |

Related: `content/maps` (same hide pattern); guest create gates unchanged on server (`requireVerifiedEditor` → `registered_user_required`); list remains `GET /api/content/packs` + client-side filter chrome only (no server filter query params).

## MODIFIED Requirements

### Requirement: Packs list filters

The packs list MUST support filters: **all** (default); **on moderation** (caller’s own open pending or needs_revision requests — packs and add-task-set as applicable); **drafts** (caller’s never-published packs without an open request, or equivalently never-published not yet submitted); **mine** (packs where the caller is `createdBy` OR has contributed at least one task set on that pack); **favorites** (packs the caller has starred). Guests and anonymous users MUST only see the public catalog subset. Filters that require identity beyond catalog (mine / favorites / moderation / drafts) MUST be **hidden from the UI** for guests (MUST NOT remain visible as disabled controls). Staff soft-unpublished visibility rules MUST still apply when staff use filters.

#### Scenario [SC-PACK-151]: Filter on moderation shows only own open items

- **GIVEN** author A has pack P pending and user B has pack Q pending
- **WHEN** A applies the on-moderation filter
- **THEN** A sees P and MUST NOT see Q solely via that filter

#### Scenario [SC-PACK-152]: Filter mine includes task-set contributor

- **GIVEN** verified user U contributed an approved or pending task set on published pack P and U is not `createdBy` of P
- **WHEN** U applies the mine filter
- **THEN** P appears for U

#### Scenario [SC-PACK-153]: Guest cannot use favorites filter meaningfully

- **GIVEN** an anonymous guest on the packs list
- **WHEN** the guest views the list
- **THEN** only in-catalog packs are shown
- **AND** starring / favorites filter MUST NOT grant a personal favorites set
- **AND** the favorites filter control MUST NOT be visible

#### Scenario [SC-PACK-286]: Guest sees only the All filter control

- **GIVEN** an anonymous guest on the packs list
- **WHEN** the filter chrome renders
- **THEN** the **all** filter control remains visible (identity filters use `v-if` / equivalent — MUST NOT remain as disabled controls)
- **AND** on-moderation, drafts, mine, and favorites filter controls are not shown
