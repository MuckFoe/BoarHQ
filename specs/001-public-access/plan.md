# 001 — Public access: plan

HOW for `spec.md`. Written 2026-10-09; revised the same day after the user
resolved the three conflicts at the end.

Inputs read: `spec.md`, `rules/*.md`, the Bindings in `CLAUDE.md`, and — beyond
what `/sdd-plan` declares — the existing `compose.yaml`,
`deploy/tailscale/cdm.json`, `deploy/caddy/Caddyfile`, `.env.example` and
`docs/infrastructure.md`. This feature changes inherited files rather than
adding new ones, so it could not be planned without them.

## Architecture

The ingress stays as it is: one Tailscale sidecar that terminates TLS and
forwards everything to Caddy on `127.0.0.1:80`, Caddy the only container in the
sidecar's network namespace, no service with `ports:`. Three things change.

### 1. The Serve handler is also published through Funnel (FR1, FR2, FR4)

Funnel is Serve's public mode: Tailscale's relays accept connections from the
internet for the node's `ts.net` name and carry them to the node over the
tunnel the node itself opened outbound. TLS is still terminated on the node
with the same publicly trusted `ts.net` certificate Serve already provisions,
so FR2 needs nothing new. Because the tunnel is outbound, no inbound port is
opened on the host and no `ports:` is added, which is what keeps FR3 and FR4
true. Tailnet members keep reaching the same name directly.

The change is one block in the Serve configuration (Contracts).

### 2. Everything named `cdm` is renamed to `boarhq` (FR1, FR5)

Nothing has ever been deployed, so this is a rename of names, not a migration:
no volume, database or realm exists under the old name.

| Was | Becomes | Where |
|---|---|---|
| compose project `cdm` | `boarhq` | `compose.yaml` `name:` |
| service `ts-cdm` | `ts-boarhq` | `compose.yaml` — the service, `network_mode: service:…` and `depends_on` on `proxy` |
| `hostname: cdm` | `hostname: boarhq` | `compose.yaml` — this is FR1's hostname |
| volume `ts-cdm-state` | `ts-boarhq-state` | `compose.yaml` — the mount and the `volumes:` entry |
| `deploy/tailscale/cdm.json` | `deploy/tailscale/boarhq.json` | moved with `git mv`; the bind mount and `TS_SERVE_CONFIG` follow |
| tag `tag:cdm` | `tag:boarhq` | `compose.yaml` `TS_EXTRA_ARGS`, `.env.example` |
| host `cdm.${TS_TAILNET}.ts.net` | `boarhq.${TS_TAILNET}.ts.net` | `compose.yaml` — `KC_HOSTNAME`, both issuer settings, `--redirect-url`, `--whitelist-domain` |
| realm `cdm` | `boarhq` | `compose.yaml` — the two issuer URLs |
| client `cdm-forward-auth` | `boarhq-forward-auth` | `compose.yaml` — `--client-id`, the audience; `.env.example` |
| `CDM_API_ORIGIN`, `CDM_LISTEN`, `CDM_DB_DSN`, `CDM_OIDC_ISSUER`, `CDM_OIDC_AUDIENCE` | `BOARHQ_…`, same suffixes | `compose.yaml` — `web` and `api` environment |
| database `cdm`, role `cdm` | `boarhq`, `boarhq` | `compose.yaml` — `db-app` environment, its health check, the DSN |
| "CDM" as the title of this system | "boarHQ" | header comments of `compose.yaml`, `deploy/caddy/Caddyfile`, `.env.example` |

The same names are corrected wherever the documents repeat them:
`docs/infrastructure.md`, `docs/compose.md`, `docs/architecture.md` (including
the diagram node id `tscdm`, the example volume `cdm_db-app-data` and the
`cdm-1` troubleshooting row), `specs/README.md` (the path in the 001 row) and
`rules/_progress.md` (the "everything is still named `cdm`" follow-up, which
this closes).

The URL settings, the realm and the client id must all land in one commit: a
partial rename is a sign-in redirect loop.

### 2b. Every trace of the inherited project is removed (FR6)

Separate from the rename: these are deletions and rewordings, not new names.

| What | Where | How |
|---|---|---|
| Sentences naming the origin — "inherited from CDM", "designed for CDM (Car Decision Model)", "CDM's design" | `specs/002`–`007` `spec.md` (Problem sections, and 006's resolved note), `specs/README.md`, `rules/_progress.md`, `docs/infrastructure.md` (title and intro), this spec's Problem section | The clause is deleted. Where the sentence carried a fact that still matters — the files predate their specs; nothing has been run — it is kept without naming a source |
| Requirement and decision identifiers: `R11`, `R12`, `R13`, `R15`, `R16`, `D2`, `D14`, `D23`–`D25`, `D27`–`D33` | comments in `compose.yaml` and `deploy/caddy/Caddyfile`; `docs/compose.md`; `docs/architecture.md`; the sentence in `docs/infrastructure.md` that lists them | The identifier is deleted and the reasoning beside it stays. Their source documents are gone, so they point at nothing |
| The inherited application's design | `docs/architecture.md` §3 Domain model, §4 Cost category matrix, §6 Re-check sequence, §7 Offer lifecycle; the depreciation and scheduler items in §8; the "robots gate" and offer-source nodes in the §1 diagram | Deleted. §5 Sign-in sequence is renumbered; the section pointer in `docs/infrastructure.md` follows |
| The same, where it leaks into the deployment document | `docs/compose.md` — the external offer-sources node and its egress edge, the "offers, observations, cost logs" volume label, the reset warning that names offers | Reworded to say "application data" |
| Link to the rendered copy of the inherited design | `docs/architecture.md` intro | The line is deleted. The published page itself is outside the repository — see Open below |
| Pointers to files that no longer exist | `docs/architecture.md` — `REQUIREMENTS.md` as a companion, `input/_analysis.md` | Deleted with the text around them |

`docs/architecture.md` loses roughly half its length. What remains — topology,
the private network, sign-in — is what describes this system.

**Exempt:** `specs/001-public-access/` — `spec.md`, this plan, `tasks.md` and
`notes.md` — because a spec that removes a name has to say which.

**Not covered — git history.** The three existing commits are pushed to
`origin/main` and carry the old name in their messages and trees. They are not
rewritten (user, 2026-10-09).

### 3. The exposure is recorded where SEC1 says it lives

| File | What changes |
|---|---|
| `CLAUDE.md` § Bindings, **Known exposures** | the entry under Contracts replaces the placeholder |
| `docs/infrastructure.md` | the intro's "never exposed to the public internet" and "everything is still named `cdm`"; § Known exposures "None" and its "enabling Funnel requires a decision" sentence; § Commands gains the operator prerequisites and the go-live rule below |
| `docs/compose.md` | the "no public ingress" diagram label and the "Funnel contradicts D14" row |
| `docs/architecture.md` | the "no public ingress" label, "Funnel … is not used", "Tailnet membership is the outer gate" |
| `compose.yaml` | the header's "Reachable ONLY from the tailnet" and its diagram line |

Each of those sentences becomes false the moment Funnel is on; they are part of
this change, not passing cleanup.

### Go-live rule

The changes are implemented and merged, but **the identity provider is not
started on a Funnel-enabled node until `003-authentication-gate` and
`004-delegated-identity` are validated** (user, 2026-10-09). Until then the only
services brought up for this feature are `ts-boarhq` and `proxy`:

```
docker compose up -d ts-boarhq proxy
```

That is enough to verify this spec (Test strategy) and exposes nothing but
Caddy answering 502. The rule is written into `docs/infrastructure.md` and
carried as an open follow-up in `rules/_progress.md` until 003 and 004 close it.

### Outside the repository — operator steps

Funnel is refused unless the tailnet allows it, and nothing in the repo can
grant that:

- MagicDNS and HTTPS certificates enabled for the tailnet (already required by
  Serve).
- The tailnet policy defines `tag:boarhq` under `tagOwners` and grants it
  Funnel: `{"target": ["tag:boarhq"], "attr": ["funnel"]}` under `nodeAttrs`.
- A new auth key tagged `tag:boarhq`, reusable off, ephemeral off, placed in
  `.env` as `TS_AUTHKEY`.
- The node name `boarhq` is free in the tailnet. A stale node of that name
  makes the new one register as `boarhq-1`, and every URL above is then wrong.

## Data model

None. This feature persists nothing new. The sidecar's state volume, under its
new name, keeps holding the node identity and certificate.

## Contracts

**Serve configuration** — `deploy/tailscale/boarhq.json`, complete file:

```json
{
  "TCP": { "443": { "HTTPS": true } },
  "Web": {
    "${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": { "Proxy": "http://127.0.0.1:80" }
      }
    }
  },
  "AllowFunnel": { "${TS_CERT_DOMAIN}:443": true }
}
```

Port 443 is fixed: Funnel accepts only 443, 8443 and 10000.

**External URL** — `https://boarhq.${TS_TAILNET}.ts.net`, with `TS_TAILNET`
still required (`:?`). Derived values, exact:

| Setting | Value |
|---|---|
| `KC_HOSTNAME` | `https://boarhq.${TS_TAILNET:?}.ts.net/auth` |
| `BOARHQ_OIDC_ISSUER`, `--oidc-issuer-url` | `https://boarhq.${TS_TAILNET:?}.ts.net/auth/realms/boarhq` |
| `BOARHQ_OIDC_AUDIENCE`, `--client-id` | `boarhq-forward-auth` |
| `--redirect-url` | `https://boarhq.${TS_TAILNET:?}.ts.net/oauth2/callback` |
| `--whitelist-domain` | `boarhq.${TS_TAILNET:?}.ts.net` |
| `BOARHQ_DB_DSN` | `postgres://boarhq:${POSTGRES_APP_PASSWORD:?}@db-app:5432/boarhq?sslmode=disable` |
| `BOARHQ_API_ORIGIN` | `http://api:8080` |
| `BOARHQ_LISTEN` | `0.0.0.0:8080` |

The `BOARHQ_*` variables are the interface the not-yet-written `web` and `api`
will read. Renaming them now costs nothing; nothing consumes them yet.

**What Caddy receives.** Unchanged: plain HTTP on `127.0.0.1:80` from the
sidecar with `X-Forwarded-Proto: https`; `trusted_proxies` stays
`127.0.0.1/32 ::1`. Only comments change in the Caddyfile. Requests arriving
through Funnel carry no Tailscale identity headers; nothing in the stack reads
them.

**Known-exposures entry** for `CLAUDE.md`:

> The whole stack is reachable from the public internet at
> `boarhq.<tailnet>.ts.net` through Tailscale Funnel (spec `001-public-access`).
> Unauthenticated: `/auth/*` (the identity provider, including its
> administration console), `/oauth2/*` (session endpoints) and `/api/healthz`.
> Everything else is behind the gate of `003-authentication-gate`.
> **Reason accepted:** club members will not install the private network's
> client (user, 2026-10-09). **What would remove it:** every user joining the
> private network, then deleting `AllowFunnel` from the Serve configuration.
> **Not yet live:** the identity provider is not started under Funnel until
> 003 and 004 are validated.

## Test strategy

No test runner, build or linter exists in this repository. The automated checks
here are one shell script, `deploy/check/001-public-access.sh`, needing only
`docker compose`, `jq` and `grep`; it exits non-zero and names what it found on
failure. It is an exception for this spec, not a pattern — later specs use real
test software (user, 2026-10-09; `rules/_decisions.md` § Structural
decisions 8).

| Criterion | Level | Check | Seen failing first (TST1) |
|---|---|---|---|
| FR3 | static, the script | Resolve the configuration (`docker compose config --format json`, with placeholder values for the required variables) and assert that no service has `ports` and none has `network_mode: host` | Passes today, so it is run once against a copy with a `ports:` entry added and must fail there |
| FR5 | static, the script | No tracked file path contains `cdm` | Fails today on `deploy/tailscale/cdm.json`; written and run before the rename |
| FR5, FR6 | static, the script | No match for `cdm` or `car decision model`, ignoring case, in any tracked file outside `specs/001-public-access/` | Fails today in fourteen files |
| FR6 | static, the script | No match for an identifier of the form `R<n>` or `D<n>` in `compose.yaml`, `deploy/` or `docs/` | Fails today on 33 lines |
| FR6 | review | `docs/architecture.md` and `docs/compose.md` are read end to end: no section, diagram node or label describes the inherited application | — |
| FR1 | deployed, manual | From a device with the private-network client off or absent (a phone on mobile data): `curl -sS -o /dev/null -w '%{http_code}' https://boarhq.<tailnet>.ts.net/` returns any HTTP status | Fails today — the name does not resolve to anything reachable off the tailnet |
| FR2 | deployed, manual | Same device, same URL in a browser with no added certificate: padlock, no warning. `curl` without `-k` succeeding covers the trust half | Fails today with FR1 |
| FR4 | deployed, manual | From the same outside device, port-scan the host's public address (`nmap -Pn -p- <address>`); no port answers with a service of this stack | Cannot be seen failing without deliberately publishing a port; it guards a regression that FR3's static check also guards |

FR1, FR2 and FR4 are verified under the go-live rule, with only `ts-boarhq` and
`proxy` running. Caddy then answers 502 for everything, and a 502 over trusted
HTTPS from an outside device satisfies FR1 and FR2 as written.

Not covered by any check, and to be said so at validation (VER5): that the
operator's tailnet policy is what the plan assumes; that nothing else on the
host machine — outside this stack — listens publicly; and that the renamed
realm, client and database actually work, which cannot be exercised until
`web`, `api` and the identity configuration of 004 exist.

## Constitution compliance

| Rule | How the plan meets it |
|---|---|
| VER1 | No fast check and no feature check exist project-wide. For this feature the gate is the script plus the manual checks (Conflict 1, resolved) |
| VER2 | No new concept. *private network* and *operator* are used by the spec and still have no entry in `docs/concepts.md`; that gap predates this plan and is already recorded in `rules/_progress.md` |
| VER3 | Nothing is suppressed |
| VER4 | Applies during implementation; the likeliest failure is a sign-in redirect loop from one missed name, and it cannot surface until 003/004 |
| VER5 | The binding is still a placeholder. The uncovered areas for this feature are named under Test strategy |
| SEC1 | The exposure, its reason and its remedy are recorded in the spec and written into the Known-exposures binding. The go-live rule keeps the unauthenticated identity endpoints off the internet until the gate is validated (Conflict 2, resolved) |
| SEC2, SEC3 | No handler and no input is added |
| SEC4 | No new secret. `TS_AUTHKEY` stays in `.env` only; the plan needs no secret's value |
| CON1 | Nothing destructive: no volume or database exists under the old names. If the operator has a local `cdm_*` volume from an experiment it is orphaned by the rename, not deleted. The deleted document sections stay recoverable from git. Rewriting git history is destructive and is outside this plan (§ 2b) |
| CON2 | Two gaps surfaced: there is no check command (Conflict 1, resolved), and the operator's tailnet policy is not described anywhere in the repo. Both are written into `docs/infrastructure.md` by this plan |
| CON3 | The rename is required by FR5 and the removal of the inherited project's traces by FR6, so both are the task, not passengers (Conflicts 3 and 4, resolved). Around a deleted identifier or clause, the remaining text is left as it was |
| DAT1, DAT2 | Nothing persisted |
| ARC1 | The established ingress pattern is kept; Funnel is a flag on the existing Serve handler, not a second ingress. `compose.yaml` and the Serve configuration are named load-bearing in `docs/infrastructure.md` — the namespace sharing and the single handler are renamed, not restructured |
| ARC2 | No dependency added |
| API1 | The external URL, the realm, the client id and the `BOARHQ_*` variables change in place. Nothing has been deployed and nothing consumes them, so there is no caller to break |
| API2 | Not applicable |
| GIT1 | Commits reference `001-public-access` |
| TST1 | Per criterion, in the Test strategy table |
| TST2 | Automated checks only on what regresses silently: a `ports:` line, a leftover name, a leftover identifier |
| STY1 | One name for the system after this change (Conflict 3, resolved) |
| STY2 | Comments that cite the tailnet-only design are rewritten to state the public one and why |

### Conflicts — all resolved 2026-10-09 (user)

1. **Nothing to run the FR3 check with (VER1, ARC1 greenfield).** A shell
   script for this spec only; later specs use real test software where
   possible. `rules/_decisions.md` § Structural decisions 8.
2. **Going public before the gate is proven (SEC1).** Implement and merge;
   verify with only the sidecar and Caddy running; do not start the identity
   provider under Funnel until 003 and 004 are validated. See Go-live rule.
3. **`boarhq` in the URL, `cdm` everywhere else (STY1 against CON3).** Rename
   everything. The spec was widened first: FR5.
4. **What the rename leaves behind.** The first revision kept prose about the
   origin project. The user removed that exception: every mention goes, history
   included. The spec was widened first: FR6. See § 2b.

### Settled, out of scope

- **Git history is not rewritten** (user, 2026-10-09). The three pushed commits
  keep the old name in their messages and trees; FR6 covers the files as they
  stand, not the log.
- **The published rendering** linked from `docs/architecture.md` is left in
  place. Only the link is removed.
