# 002 — Single-hostname routing

## Problem / Why

Every service in the stack is reached under one hostname and told apart by URL
path. One hostname means one certificate, one sign-in session shared by all
services, and no per-service address for a club member to remember. Adding a service
must not disturb the ones already there. Inherited from the CDM infrastructure
(`docs/infrastructure.md`).

## Users

- **Club member** — reaches every service through the same hostname.
- **Operator** — adds and removes services.

## Functional requirements

- **FR1** — THE SYSTEM SHALL serve every service under one hostname,
  distinguished by URL path.
- **FR2** — WHEN a request's path matches no service-specific route THE SYSTEM
  SHALL deliver it to the web frontend.
- **FR3** — WHEN a service is added under a new path THE SYSTEM SHALL keep every
  existing service's URL working unchanged.
- **FR4** — WHEN the service behind a route is not running THE SYSTEM SHALL
  return an error for that route and continue serving all other routes.

## Non-functional requirements

- None stated.

## Out of scope

- Whether a request is authenticated before it is routed — see
  `003-authentication-gate`.
- What each service does once the request reaches it.
- Services that need a hostname of their own. They are excluded, not
  accommodated.

## Acceptance criteria

- **FR1** — Given a running stack, when a club member requests each service's path
  under the one hostname, then each request is answered by that service.
- **FR2** — Given a signed-in club member, when they request a path no service
  claims, then the web frontend answers.
- **FR3** — Given a running stack, when a service is added under a new path,
  then every URL that responded before still responds the same way.
- **FR4** — Given one service is stopped, when a club member requests its path, then
  they receive an error; and when they request any other service's path, then
  that service answers normally.

## Open questions

- Resolved 2026-10-09: boarHQ has one web frontend and one API for now. More
  services, and more data stores, may be added later; FR3 is what keeps that
  from disturbing the existing ones.
- Resolved 2026-10-09: FR1 has no exceptions. A service that cannot run under a
  path of the one hostname is not added to the stack; it does not get a
  hostname of its own.
