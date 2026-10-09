# 006 — Secrets and deploy-time configuration

## Problem / Why

The stack needs secrets to run — a key to join the private network, database
passwords, the identity provider's first administrator password, and the secrets
of the component that checks sessions. None of them may be in the repository,
and a stack started without one must fail visibly instead of running on a
guessable default. Inherited from the CDM infrastructure
(`docs/infrastructure.md`).

## Users

- **Operator** — supplies the values before the first start.

## Functional requirements

- **FR1** — THE SYSTEM SHALL read every secret from deploy-time configuration
  that is excluded from version control.
- **FR2** — WHEN a required secret or setting is missing at start THE SYSTEM
  SHALL refuse to start and name the missing value.
- **FR3** — THE SYSTEM SHALL NOT fall back to a default value for any secret.
- **FR4** — THE SYSTEM SHALL provide, under version control, a template that
  lists every required setting without its value.

## Non-functional requirements

- SEC4: secrets never appear in source, committed configuration, logs, error
  messages or agent output.

## Out of scope

- Rotating secrets.
- Settings that belong to a particular application feature and not to the
  infrastructure.

## Acceptance criteria

- **FR1** — Given the repository, when its tracked files are inspected, then
  none contains a secret value, and the file that holds them is excluded from
  version control.
- **FR2** — Given deploy-time configuration with one required value removed,
  when the stack is started, then it does not start and the output names that
  value.
- **FR3** — Given the deployment configuration, when it is inspected, then no
  secret has a default value.
- **FR4** — Given a fresh copy of the repository, when the operator fills in
  every entry of the template, then the stack has every setting it requires.

## Open questions

None. Resolved 2026-10-09: the CDM application settings (re-check interval,
per-source overrides, fetch user-agent) do not belong to boarHQ and were removed
from the template and the API's settings the same day.
