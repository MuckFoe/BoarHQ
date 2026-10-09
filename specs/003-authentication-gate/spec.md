# 003 — Authentication gate

## Problem / Why

No application service should have to decide whether a request is
authenticated. Authentication is enforced once, at the entry point, for every
request, so a service added later is protected without writing any sign-in code.
The API must still not rely on that alone: other services inside the stack can
reach it directly without passing the entry point, so it checks a signed proof
of identity for itself. Inherited from the CDM infrastructure
(`docs/infrastructure.md`).

## Users

- **Club member** — signs in once and uses every service.
- **Application service** — receives requests that are already authenticated.

## Functional requirements

- **FR1** — WHEN a request without a valid session arrives for a browser-facing
  route THE SYSTEM SHALL redirect it to sign-in and, after a successful sign-in,
  return the club member to the URL originally requested.
- **FR2** — WHEN a request without a valid session arrives for an API route THE
  SYSTEM SHALL respond with status 401 and SHALL NOT redirect.
- **FR3** — WHEN a request with a valid session arrives THE SYSTEM SHALL forward
  it to the target service together with the club member's identity and a signed
  proof of that identity.
- **FR4** — WHEN a request carries identity information supplied by the client
  THE SYSTEM SHALL discard that information before authenticating the request.
- **FR5** — THE SYSTEM SHALL leave exactly three kinds of route ungated: the
  identity provider's own endpoints, the endpoints that establish a session, and
  the API's liveness probe.
- **FR6** — WHEN the liveness probe is requested THE SYSTEM SHALL answer without
  revealing configuration, version information or data.
- **FR7** — WHEN a request reaches the API without a valid signed proof of
  identity THE SYSTEM SHALL reject it, whichever path it arrived by. Valid means
  the signature, issuer, audience and expiry all check out.

## Non-functional requirements

- Fail closed: when the component that checks sessions is unavailable, every
  gated route is refused. Only the ungated routes of FR5 keep answering.

## Out of scope

- Authorisation — what a signed-in club member is allowed to do.
- How accounts come to exist — see `004-delegated-identity`.
- Network-level reachability — see `001-public-access`.

## Acceptance criteria

- **FR1** — Given a club member with no session, when they open a browser-facing URL,
  then they are sent to sign-in; and when they sign in, then they land on the
  URL they first asked for.
- **FR2** — Given a client with no session, when it calls an API route, then the
  response is 401 with no redirect and no sign-in page in the body.
- **FR3** — Given a signed-in club member, when they request a gated route, then the
  target service receives the club member's identity and a signed proof of it.
- **FR4** — Given a client with no session that sends a request claiming to be
  some club member, when the request arrives, then it is treated as unauthenticated
  and the claim never reaches a service.
- **FR5** — Given a client with no session, when it requests a route of each of
  the three ungated kinds, then each answers; and when it requests any other
  route, then FR1 or FR2 applies.
- **FR6** — Given a client with no session, when it requests the liveness probe,
  then the response contains no configuration, version or data.
- **FR7** — Given a request sent to the API from inside the stack, bypassing the
  entry point, when it carries no proof, or a proof that is expired, wrongly
  signed, or issued for a different audience or by a different issuer, then the
  API rejects it.

## Open questions

None. Resolved 2026-10-09: nothing in boarHQ is reachable without signing in
beyond the three kinds of route in FR5. No public landing page.
