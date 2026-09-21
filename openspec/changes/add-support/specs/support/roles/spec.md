## Purpose

Роли пользователей для поддержки: `user` (по умолчанию), `moderator`, `admin`. Первый admin через env id; только admin назначает роли через список пользователей. Moderator обрабатывает тикеты, но не назначает admin.

## Traceability

| Scenario ID | Coverage |
|-------------|----------|
| SC-ROLE-01 | pending (server: default role user) |
| SC-ROLE-02 | pending (server: BOOTSTRAP_ADMIN_IDS idempotent) |
| SC-ROLE-03 | pending (server: moderator cannot set admin) |
| SC-ROLE-04 | pending (server: admin can set moderator/admin/user) |
| SC-ROLE-05 | pending (server: non-admin cannot change roles) |
| SC-ROLE-06 | pending (server: staff endpoints require moderator+) |
| SC-ROLE-07 | pending (server: admin user list) |
| SC-ROLE-08 | pending (client: admin users UI gated) |

## ADDED Requirements

### Requirement: Every user has a role defaulting to user

Each persisted auth user MUST have a role of exactly one of: `user`, `moderator`, `admin`. New users MUST default to `user` unless elevated by bootstrap or an admin.

#### Scenario [SC-ROLE-01]: New user defaults to user role

- **GIVEN** a newly registered or anonymous user row
- **WHEN** the account is created without bootstrap elevation
- **THEN** the user's role is `user`

### Requirement: Bootstrap admin ids from environment are held as admin

On server start, user ids listed in the bootstrap-admin environment setting MUST be set to role `admin` idempotently (re-applied on each start so env wins over a manual demotion of those ids).

#### Scenario [SC-ROLE-02]: Bootstrap env ids become admin on start

- **GIVEN** a registered user id listed in the bootstrap-admin environment setting
- **AND** that user's role is currently `user` or `moderator`
- **WHEN** the server starts (or bootstrap runs)
- **THEN** that user's role is `admin`

### Requirement: Only admin may change roles; moderator cannot grant admin

Changing another user's role MUST require the actor to be `admin`. An admin MUST be able to set a target to `user`, `moderator`, or `admin`. A moderator MUST NOT change roles. A non-admin MUST NOT access the admin user-list mutation API.

#### Scenario [SC-ROLE-03]: Moderator cannot assign admin

- **GIVEN** an actor with role `moderator`
- **WHEN** the actor attempts to set another user's role to `admin` (or any role change)
- **THEN** the system rejects the request
- **AND** the target role is unchanged

#### Scenario [SC-ROLE-04]: Admin can assign moderator or admin

- **GIVEN** an actor with role `admin`
- **WHEN** the actor sets another user's role to `moderator` or `admin` or `user`
- **THEN** the target user's role becomes the requested value

#### Scenario [SC-ROLE-05]: Regular user cannot change roles

- **GIVEN** an actor with role `user`
- **WHEN** the actor attempts to change any user's role
- **THEN** the system rejects the request

### Requirement: Staff ticket actions require moderator or admin

Endpoints that list all tickets, take tickets into work, or post as staff MUST require role `moderator` or `admin`. Role `user` MUST be rejected.

#### Scenario [SC-ROLE-06]: User cannot access staff ticket actions

- **GIVEN** an actor with role `user`
- **WHEN** the actor calls a staff-only support action (list all / take into work / staff status change)
- **THEN** the system rejects the request

### Requirement: Admin can list users to assign roles

An admin MUST be able to retrieve a user list suitable for assigning roles (at least identity and current role). The client MUST expose this admin-only users UI; moderators MUST NOT get role-assignment UI.

#### Scenario [SC-ROLE-07]: Admin receives user list

- **GIVEN** an actor with role `admin`
- **WHEN** the actor requests the admin user list
- **THEN** the response includes users with their roles

#### Scenario [SC-ROLE-08]: Role assignment UI is admin-only

- **GIVEN** an actor with role `moderator` (not admin)
- **WHEN** the actor uses the client support/staff area
- **THEN** the role-assignment users screen is not available as an admin capability
- **AND** staff ticket tools remain available
