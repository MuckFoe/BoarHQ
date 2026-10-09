# 005 — Persistence

## Problem / Why

Stopping, restarting or rebuilding the stack must not lose anything. Application
data and identity data are also kept apart, so one can be reset, dumped or
restored without touching the other. boarHQ's application data exists nowhere
else, so losing it is permanent (user, 2026-10-09). Inherited from the CDM infrastructure
(`docs/infrastructure.md`).

## Users

- **Operator** — stops, rebuilds and occasionally resets the stack.
- **Club member** — expects their data and their account to still be there afterwards.

## Functional requirements

- **FR1** — WHEN the stack is stopped and started again, or rebuilt, THE SYSTEM
  SHALL retain application data, identity data, and the stack's identity on the
  private network.
- **FR2** — THE SYSTEM SHALL keep application data and identity data in separate
  stores.
- **FR3** — WHEN the operator resets one of the two stores THE SYSTEM SHALL
  leave the other intact.
- **FR4** — THE SYSTEM SHALL lose persisted state only through an operation the
  operator invokes explicitly for that purpose.
- **FR5** — THE SYSTEM SHALL back up application data and identity data once
  every day without operator action.
- **FR6** — WHEN the operator restores a backup THE SYSTEM SHALL return that
  store to its state at the time the backup was taken.
- **FR7** — THE SYSTEM SHALL keep each daily backup for 14 days and SHALL
  remove it after that.

## Non-functional requirements

- Destroying persisted state is a destructive data operation under CON1: it is
  confirmed first, stating what is destroyed and that there is no recovery path.

## Out of scope

- The shape of the data stored — that belongs to the features that store it.
- Backups stored anywhere other than the machine boarHQ runs on. Until that
  changes, losing the machine or its disk loses the backups with the data.

## Acceptance criteria

- **FR1** — Given a stack holding application data and at least one account,
  when it is stopped and started again, or rebuilt, then the data and the
  account are still present and the stack answers on the same private-network
  identity as before.
- **FR2** — Given the deployment configuration, when it is inspected, then
  application data and identity data are in different stores.
- **FR3** — Given a stack holding application data and accounts, when the
  operator resets the application store, then the accounts still work; and when
  the operator resets the identity store, then the application data is still
  present.
- **FR4** — Given a running stack, when it is stopped, restarted, rebuilt, or a
  service crashes, then no persisted state is lost.
- **FR5** — Given a stack that has run untouched for two days, when the backups
  are listed, then there is one of application data and one of identity data
  for each of those days.
- **FR6** — Given a backup and data changed since it was taken, when the
  operator restores that backup, then the store holds exactly what it held when
  the backup was taken.
- **FR7** — Given a stack that has run for more than 14 days, when the backups
  are listed, then the last 14 days are present for each store and nothing
  older is.

## Open questions

- Resolved 2026-10-09: the application data boarHQ stores exists nowhere else.
  If it is lost it cannot be recreated. What the data is has not been stated;
  it belongs to the features that store it.
- Resolved 2026-10-09: backup is required and runs daily (FR5, FR6).
- Resolved 2026-10-09: backups are kept for 14 days (FR7).
- Resolved 2026-10-09: backups are stored on the machine boarHQ runs on, for
  now. Storing them elsewhere is intended later and is not part of this spec.

None open.
