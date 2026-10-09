# 004 — Delegated identity

## Problem / Why

The application must never handle a credential. Accounts, passwords and sign-in
live in a dedicated identity provider that runs as its own service, so a defect
in application code cannot leak or mishandle a password. Inherited from the CDM
infrastructure (`docs/infrastructure.md`).

## Users

- **Club member** — registers for an account and, once approved, signs in at
  the identity provider.
- **Administrator** — approves accounts in the identity provider.

## Functional requirements

- **FR1** — THE SYSTEM SHALL delegate all credential handling to a dedicated
  identity provider; no application service SHALL receive, store or verify a
  password.
- **FR2** — THE SYSTEM SHALL serve the identity provider under the same hostname
  as every other service.
- **FR3** — WHEN the stack starts for the first time THE SYSTEM SHALL create one
  administrator account for the identity provider from deploy-time
  configuration.
- **FR4** — WHEN a visitor registers at the sign-in page THE SYSTEM SHALL create
  an account for them that is not yet approved.
- **FR5** — WHEN the holder of an account that is not approved attempts to sign
  in THE SYSTEM SHALL refuse and SHALL NOT establish a session.
- **FR6** — WHEN an administrator approves an account THE SYSTEM SHALL allow its
  holder to sign in from then on.
- **FR7** — WHEN one week has passed since a club member signed in THE SYSTEM
  SHALL end their session and require them to sign in again.
- **FR8** — WHEN the stack starts for the first time THE SYSTEM SHALL configure
  the identity provider from a definition kept under version control, with no
  manual configuration step.

## Non-functional requirements

- The version-controlled definition contains no secret (SEC4); secrets it needs
  come from deploy-time configuration — see `006-secrets-and-configuration`.

- The identity provider's data is stored apart from application data — see
  `005-persistence`.

## Out of scope

- How a session is checked on each request — see `003-authentication-gate`.
- Signing in with an account from another provider.
- What happens to an account an administrator declines, and how an
  administrator learns that one is waiting. Not stated.

## Acceptance criteria

- **FR1** — Given a member signing in, when the sign-in completes, then no
  application service has received the password at any point.
- **FR2** — Given a member with no session, when they are sent to sign-in, then
  the sign-in page is served under the stack's one hostname.
- **FR3** — Given a stack that has never been started, when it starts with the
  administrator settings supplied, then the operator can sign in to the identity
  provider's administration with them.
- **FR4** — Given a visitor with no account, when they register at the sign-in
  page, then an account exists for them and it is not approved.
- **FR5** — Given an account that is not approved, when its holder signs in with
  the correct password, then sign-in is refused and no gated route answers them.
- **FR6** — Given an account that is not approved, when an administrator
  approves it, then its holder can sign in and gated routes answer them.
- **FR7** — Given a club member who signed in less than a week ago, when they
  request a gated route, then it answers without a new sign-in; and given one
  who signed in more than a week ago, when they request a gated route, then
  they are sent to sign-in.
- **FR8** — Given a stack that has never been started and a fresh copy of the
  repository, when it starts, then registration, approval and the one-week
  session behave as FR4–FR7 require without anyone having configured the
  identity provider by hand.

## Open questions

- Resolved 2026-10-09: accounts come from self-registration, and an account
  cannot be used until an administrator has approved it.
- Resolved 2026-10-09: a session lasts one week, counted from sign-in. Activity
  does not extend it.
- Resolved 2026-10-09: the identity provider's configuration is kept as a file
  in the repository (FR8).

None open.
