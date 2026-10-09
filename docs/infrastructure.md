# Infrastructure — inherited from CDM

This deployment topology was designed for CDM (Car Decision Model) and merged
into boarHQ on 2026-10-09 after the CDM application design was scrapped: a
private stack reached over Tailscale, never exposed to the public internet.
Everything is still named `cdm` — compose project, hostname, database, Keycloak
realm, `CDM_*` variables. Comments in `compose.yaml`, the Caddyfile and the docs
still cite CDM requirement and decision ids (R11, R15, R16, D23, D30, D32, D33)
whose source documents are gone; they do not refer to boarHQ's `REQUIREMENTS.md`.

## Stack

- **api** `api/` and **web** `web/` — referenced by `compose.yaml`, *not
  created*. `docker compose up` fails at build until they exist.
- **auth** — Keycloak 26, own container, own database, served under `/auth`.
- **authproxy** — oauth2-proxy. Gates **every** request: Caddy redirects an
  unauthenticated browser here, and passes `api` the Bearer token it returns.
- **Data** — Postgres 17, two instances: `db-app` and `db-auth`.
- **Ingress** — a Tailscale sidecar running Serve, plus **Caddy** as the reverse
  proxy. Serve terminates TLS and forwards everything to Caddy; Caddy routes.

One hostname, one Serve handler, all routing in the Caddyfile:
`/auth/*` → Keycloak :8080, `/oauth2/*` → authproxy :4180,
`/api/*` → api :8080, everything else → web :3000.

**Only `proxy` uses `network_mode: service:ts-cdm`.** Tailscale Serve can proxy
only to `127.0.0.1`, so one container must share the sidecar's namespace — that
container is Caddy. Every other service is ordinary, keeps its natural port, and
is reached by DNS name. Adding a service is a Caddyfile block and nothing else.

## Commands

| Task | Command |
|---|---|
| Bring the stack up | `docker compose up -d` |
| Follow logs | `docker compose logs -f api web auth` |
| Tear down, keep data | `docker compose down` |
| Tear down, **destroy data** | `docker compose down -v` — confirm first |

**Prerequisite:** `.env` must exist, copied from `.env.example`, with a
Tailscale auth key, the tailnet name and the database passwords. Compose refuses
to start without them rather than falling back to a default. Secrets never enter
the repo, logs, or agent output.

No compiler invocation, test runner or linter exists here. Never report a change
as verified; say that nothing could be checked.

## Read before touching

| When you are... | Read |
|---|---|
| touching `compose.yaml`, ingress, or deployment | `docs/compose.md` — the namespace mechanism will bite otherwise |
| working out how the pieces fit | `docs/architecture.md` §1, §2, §5 — the other sections describe the removed CDM domain model |

## Load-bearing

Do not refactor these casually:

- `compose.yaml` — `network_mode: service:ts-*` shares kernel network
  namespaces. "Tidying" it into ordinary bridge networking silently breaks
  Tailscale Serve, and with it TLS, in a way that looks like a certificate
  problem rather than a topology one.
- `deploy/tailscale/cdm.json` — the ports here must match what `web`, `api` and
  `auth` actually bind. Nothing checks this; a mismatch is a 502 with no
  explanation. `KC_HOSTNAME` and `KC_HTTP_RELATIVE_PATH` must likewise agree
  with the `/auth/` handler, or login enters a redirect loop.
- `deploy/caddy/Caddyfile` — Caddy must strip client-supplied
  `X-Auth-Request-*` and `Authorization` before `forward_auth`; without it a
  request can assert its own identity and be believed. `/api/*` must never
  redirect on 401, which is why there are two snippets, `(sso)` and
  `(sso-api)`. `trusted_proxies` keeps Serve's `X-Forwarded-Proto: https`
  intact; remove it and Keycloak login loops.

## Known exposures

**None.** No service in `compose.yaml` declares `ports:`, so nothing is
published on a host interface, and the only ingress is a WireGuard tunnel that
requires tailnet membership. Two changes would create an exposure and both
should be treated as requiring a decision: adding `ports:` to any service, or
enabling Tailscale **Funnel**, which publishes to the public internet.
