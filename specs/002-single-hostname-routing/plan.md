# 002 — Single-hostname routing: plan

HOW for `spec.md`. Written 2026-10-09.

Inputs read: `spec.md`, `rules/*.md`, the Bindings in `CLAUDE.md`, and — beyond
what `/sdd-plan` declares — the existing `deploy/caddy/Caddyfile`, the `proxy`
service in `compose.yaml` and `docs/compose.md` §6 and §10. The routing this
spec describes already exists in those files; the plan is the difference
between them and the spec.

## Architecture

The proxy (Caddy) is the one router. It is the only thing the ingress forwards
to, it matches on path, and it hands each request to one service by that
service's name on the compose network. That stays. FR1 and FR2 are already what
`deploy/caddy/Caddyfile` does:

| Path | Service | Requirement |
|---|---|---|
| `/auth/*` | `auth:8080` | FR1 |
| `/oauth2/*` | `authproxy:4180` | FR1 |
| `/api/healthz`, `/api/*` | `api:8080` | FR1 |
| anything else | `web:3000` | FR2 |

Three things change.

### 1. A service is added as a file, not as an edit (FR3)

Today a new service means editing the Caddyfile between the API route and the
catch-all. The Caddyfile is load-bearing (`docs/infrastructure.md`), and an
edit there is exactly how an existing route gets disturbed. Instead:

- `deploy/caddy/routes/` holds one file per added service, `<name>.caddy`.
- The site block imports them where the "future services" comment is now:
  `import routes/*.caddy`. A glob that matches nothing is not an error, so the
  directory starts with no route files.
- `compose.yaml` mounts the directory on `proxy`, read-only, at
  `/etc/caddy/routes`, beside the existing Caddyfile mount.
- `deploy/caddy/routes/README.md` states the contract (Contracts). It keeps the
  directory in git and is not matched by the glob.

The four built-in routes stay in the Caddyfile: they are the stack, not
additions to it. Caddy orders `handle` blocks by how specific their path is, not
by where they appear, so an imported route cannot fall behind the catch-all.

### 2. The advice to give a service its own hostname is removed (FR1)

The spec rules it out. Two places still recommend it:

- `deploy/caddy/Caddyfile` — the "Future services go here" comment block is
  replaced whole by the import and a two-line comment pointing at
  `routes/README.md`.
- `docs/compose.md` §10 — the row "A service that cannot run under a sub-path"
  now says it is not added to the stack.

`docs/compose.md` §6 "Adding a service", its §10 row "Adding a service", and
the sentence in `docs/infrastructure.md` § Stack that says adding a service is
"a Caddyfile block" are corrected to say "a file in `deploy/caddy/routes/`".

### 3. A routing test stack (FR1–FR4)

Routing can be proven without the application, which does not exist yet: the
real Caddyfile is run against stand-ins that answer with their own name.

```
deploy/test/
  compose.test.yaml          the proxy with the real Caddyfile and routes/,
                             plus one stand-in per upstream
  compose.test.extra.yaml    override: adds one more stand-in and mounts
                             fixtures/routes/ over routes/
  stub.Caddyfile             the stand-in: answers on its service's port
                             with its own name
  fixtures/routes/extra.caddy   a route file for the FR3 test
  routing.test.mjs           the tests
```

In the test stack the proxy is an ordinary service on the bridge network — no
ingress sidecar, no shared namespace — and publishes its port 80 on a random
loopback port so the tests can reach it. That publication is in the test file
only; `compose.yaml` is untouched by it.

## Data model

None. Routing persists nothing.

## Contracts

**Route file** — `deploy/caddy/routes/<name>.caddy`, one per added service:

```caddyfile
handle /<name>/* {
	import sso
	reverse_proxy <service>:<port>
}
```

- Exactly one `handle` block; the path is `/<name>/*` and `<name>` is the file
  name.
- `<name>` is not `auth`, `oauth2` or `api`, and is not a prefix of or prefixed
  by another route's name.
- `<service>` is the compose service name; the port is the one the service
  binds, unchanged.
- The path prefix is passed through, not stripped: the service must serve under
  `/<name>/`. One that cannot is not added (spec, Open questions).
- Whether the block imports `sso`, `sso-api` or nothing is decided by
  `003-authentication-gate`, not here.

**Import point** — in the site block of `deploy/caddy/Caddyfile`, between the
`/api/*` route and the catch-all:

```caddyfile
	# Added services: one file each in routes/. See routes/README.md.
	import routes/*.caddy
```

**Mount** — on `proxy` in `compose.yaml`:

```yaml
      - ./deploy/caddy/routes:/etc/caddy/routes:ro
```

**Error for a route whose service is down (FR4)** — status `502`, Caddy's
default empty body. No upstream name, address or error text reaches the client.

**Stand-in** — `stub.Caddyfile`, parameterised by two environment variables:

| Variable | Meaning |
|---|---|
| `STUB_NAME` | the service it stands in for: `web`, `api`, `auth`, `authproxy`, `extra` |
| `STUB_PORT` | the port that service binds: 3000, 8080, 8080, 4180, 9000 |

Every response carries `X-Stub: <STUB_NAME>` and status 200, except
`/oauth2/auth`, which answers 202 so the gate lets every request through. The
tests assert on `X-Stub` and on status only.

**Test command** — `node --test deploy/test/`. Needs Docker running and Node
18 or later; no `npm install`, no dependency. It goes into the Commands table
of `CLAUDE.md` as the first real entry there.

## Test strategy

One level: integration, against the real Caddyfile. There is no logic here to
unit-test — the Caddyfile is the logic.

| Criterion | Test | Seen failing first (TST1) |
|---|---|---|
| FR1 | With the base stack up: `/auth/x` → `auth`, `/oauth2/x` → `authproxy`, `/api/healthz` → `api`, `/api/x` → `api` | Written first against a stack whose stand-ins all answer as `web`; every assertion but the last fails |
| FR2 | `/`, `/anything`, `/a/deep/path` → `web` | Same run, with the `web` stand-in misnamed |
| FR3 | Record the answering service and status for every URL of FR1 and FR2. Bring the stack up again with `compose.test.extra.yaml`. Assert the record is identical, and `/extra/x` → `extra` | Fails before the `import` line exists: `/extra/x` → `web` |
| FR4 | Stop `api`: `/api/x` → 502; `/` → `web`; `/auth/x` → `auth`. Start it, stop `web`: `/` → 502; `/api/x` → `api` | Fails if the proxy is configured to wait for upstreams at start; observed by asserting 200 first and watching it fail |
| FR4, shared gate | Stop `authproxy`: `/oauth2/x` → 502 — and so does every gated route. Asserted as it is, pending Conflict 2 | — |

The tests fail, not skip, when Docker is not running (VER3).

Not covered, and to be said so at validation (VER5):

- The real `web` and `api`. The stand-ins prove the routing table, not that a
  real service works under its path — a frontend that builds absolute URLs
  from the wrong base still breaks.
- The ingress in front of the proxy. The test stack has none.
- FR3 for route files nobody has written yet. The test proves the mechanism
  with one fixture; a future route file that breaks the naming contract is
  caught only by reading it.

## Constitution compliance

| Rule | How the plan meets it |
|---|---|
| VER1 | `node --test deploy/test/` is the feature check for this spec and the first entry in the Commands table. There is still no fast check; a Caddyfile has no build |
| VER2 | No new domain concept. *Route* is used in its ordinary sense |
| VER3 | No skipping when Docker is absent |
| VER4 | Applies during implementation |
| VER5 | The binding is still a placeholder; the uncovered areas are named above |
| SEC1 | No new exposure. The test stack publishes on loopback only, with stand-ins that hold no data |
| SEC2 | The 502 carries no internal detail |
| SEC3 | No input is accepted |
| SEC4 | The test stack needs no secret and reads no `.env` |
| CON1 | Nothing destructive. The test stack is brought down with its own project name and has no volumes |
| CON2 | The missing check command is filled in by this plan |
| CON3 | The Caddyfile change is the comment block and one `import`. Routes, snippets and global options are untouched |
| DAT1, DAT2 | Nothing persisted |
| ARC1 | **Conflicts 1 and 3** |
| ARC2 | The proxy depends on services by name; no service knows the proxy's routes |
| API1 | No URL changes. Preserving them is FR3 |
| API2 | Not applicable |
| GIT1 | Commits reference `002-single-hostname-routing` |
| TST1 | Per criterion, in the table |
| TST2 | Tests cover the routing table and its failure behaviour — the two things that break. The stand-ins themselves are not tested |
| STY1 | "Route" for a path-to-service mapping, "route file" for its file, "stand-in" for a test upstream; each used one way |
| STY2 | The new Caddyfile comment says where routes live and why not here |

### Conflicts for the human

1. **Which test software (ARC1 greenfield; `rules/_decisions.md` § Structural
   decisions 8).** This is the first plan to need one, so it proposes it, and
   the choice is the precedent for 003–007.
   - *Suggested:* Node's built-in test runner. Already installed, no
     dependency, and one program both drives the containers and makes the HTTP
     assertions. Cost: the test stack must publish a loopback port.
   - *Alternative:* Hurl, run as a container inside the test network.
     Declarative and needs no published port. Cost: it cannot start or stop
     containers, so FR3 and FR4 need a wrapper script around it — the thing
     decision 8 moved away from.
   - *Alternative:* Go's test runner. Installed, and a natural fit if the API
     turns out to be written in Go. Cost: that is undecided, and it would put a
     Go module in the repository before any Go code.

2. **FR4 cannot hold for the session-checking service.** It sits on the path
   of every gated route by design (`003-authentication-gate`), so when it is
   down, the API and the web frontend answer 502 too — not "all other routes
   continue". The same is true, more slowly, of the identity provider. Either
   FR4 is read as applying to application services and the spec says so, or
   the spec keeps FR4 as written and this is a known failure of it. The plan
   does not pick; the test asserts today's behaviour.

3. **A routes directory instead of one file.** It changes how the load-bearing
   Caddyfile is extended. The reason is FR3: an added service never opens the
   file the existing routes live in, and the test exercises the same mechanism
   an operator uses. Alternative: keep the single file and test FR3 by
   generating a modified copy — no structural change, but the test then proves
   something about a file nobody deploys.
