# 001 — Public access

## Problem / Why

boarHQ is reachable from the public internet: anyone with the URL reaches it
without joining the operator's private network. Its users are club members who
will not install the private network's client (user, 2026-10-09). The private
network is how the stack is published, not who may use it. Signing in is
therefore the only barrier between the internet and the application — see
`003-authentication-gate`. The host machine itself still publishes nothing; the
one published entry point is the only way in.

This reverses the inherited CDM infrastructure, which was private-network-only
(`docs/infrastructure.md`).

## Users

- **Club member** — the intended user; reaches boarHQ from an ordinary device
  with no private-network client installed.
- **Visitor** — anyone else on the internet who has the URL.
- **Operator** — the person who deploys the stack.

## Functional requirements

- **FR1** — THE SYSTEM SHALL be reachable from the public internet at one
  hostname, named `boarhq`, without the visitor's device being a member of the operator's
  private network.
- **FR2** — WHEN a visitor connects THE SYSTEM SHALL serve them over HTTPS with
  a certificate their device trusts without a manual exception.
- **FR3** — THE SYSTEM SHALL NOT make any service reachable through a network
  interface of the host machine.
- **FR4** — THE SYSTEM SHALL admit inbound traffic through the one published
  entry point only.

## Non-functional requirements

- Public reachability is an exposure decision under SEC1. Reason accepted: club
  members will not install the private network's client (user, 2026-10-09).
  What would remove it: every user joining the private network. The routes left
  ungated by `003-authentication-gate` — the identity provider's endpoints
  including its administration, the session endpoints, and the liveness probe —
  are reachable by anyone on the internet.

## Out of scope

- Who a visitor is and what they may do — see `003-authentication-gate`.
- Outbound connections made by services.
- Protection against abuse of the public entry point: rate limiting, lockout,
  denial of service.

## Acceptance criteria

- **FR1** — Given a device that is not a member of the operator's private
  network, when it requests the stack's hostname, then it receives a response.
- **FR2** — Given such a device with no manually installed certificate, when it
  opens the stack's hostname in a browser, then the page loads over HTTPS with
  no certificate warning.
- **FR3** — Given the resolved deployment configuration, when it is inspected,
  then no service publishes a port on the host.
- **FR4** — Given a device on the internet, when it requests the host machine's
  own address on any port, then no service of the stack responds.

## Open questions

None. Resolved 2026-10-09:

- Public, not private-network-only.
- Reason for the exposure: club members will not install the private network's
  client.
- The hostname is the name `boarhq` under the private network provider's
  domain. A custom domain is not part of this spec.
