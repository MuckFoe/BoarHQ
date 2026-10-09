# 007 — Startup and recovery

## Problem / Why

The stack is started with one command and left alone. Services must come up in
an order that works, and a service that crashes must come back without the
operator noticing — but a service the operator stopped on purpose must stay
stopped. Inherited from the CDM infrastructure (`docs/infrastructure.md`).

## Users

- **Operator** — starts the stack and otherwise does not attend to it.

## Functional requirements

- **FR1** — WHEN the stack starts THE SYSTEM SHALL start each service that needs
  a database only after that database reports ready.
- **FR2** — WHEN a service's process exits unexpectedly THE SYSTEM SHALL restart
  it automatically.
- **FR3** — WHEN the operator stops a service deliberately THE SYSTEM SHALL keep
  it stopped until it is explicitly started again, including across a restart of
  the host machine.
- **FR4** — WHEN a service is not yet available THE SYSTEM SHALL return an error
  for requests to it and SHALL serve them without operator action once it is.

## Non-functional requirements

- None stated. There is no limit on restart attempts and no alerting; this is
  an accepted gap, not an oversight (user, 2026-10-09).

## Out of scope

- Monitoring and alerting. A WhatsApp or Signal notification to the operator is
  a future feature with its own spec.
- Upgrading service versions.

## Acceptance criteria

- **FR1** — Given a stack that is not running, when it is started, then no
  service that needs a database starts before that database reports ready.
- **FR2** — Given a running stack, when a service's process is killed, then that
  service is running again without operator action.
- **FR3** — Given a service the operator has stopped, when the host machine
  restarts, then that service is still stopped.
- **FR4** — Given a stack that is still starting, when a club member requests a
  service that is not up yet, then they receive an error; and when the service
  has come up, then the same request succeeds with nothing restarted by hand.

## Open questions

- Resolved 2026-10-09: accepted for now that a crash-looping service retries
  forever and nobody is told. A notification to the operator over WhatsApp or
  Signal is intended later and is not part of this spec.
- Resolved 2026-10-09: accepted that the stack reports healthy about a minute
  before its hostname answers, while the certificate is issued.

None open.
