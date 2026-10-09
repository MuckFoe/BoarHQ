# 001 — Public access: tasks

From `plan.md` and `spec.md`, 2026-10-09. One task, or one `[P]` wave, per
`/sdd-implement` pass. Commits reference `001-public-access` (GIT1).

"The script" is `deploy/check/001-public-access.sh`, written in task 1. It has
four checks, named here so later tasks can cite them:

- **ports** — no service in the resolved configuration has `ports` or
  `network_mode: host` (FR3).
- **paths** — no tracked file path contains `cdm` (FR5).
- **names** — no match for `cdm` or `car decision model`, ignoring case, in any
  tracked file outside `specs/001-public-access/` (FR5, FR6).
- **ids** — no identifier of the form `R<n>` or `D<n>` in `compose.yaml`,
  `deploy/` or `docs/` (FR6).

**Feature gate:** after tasks 2–10 the script exits 0 with all four checks
passing. Task 11 is the part no script can do.

## Waves

| Wave | Tasks | Why this order |
|---|---|---|
| 1 | 1 | The checks exist and are seen failing before anything is changed (TST1) |
| 2 | 2 | The rename moves a file and must be one commit; everything else builds on the new names |
| 3 | 3, 4, 5, 6, 7, 8, 9, 10 — all `[P]` | No two touch the same file |
| 4 | 11 | Needs everything merged, and the operator |

## Checklist

Tick a task here when it is done; the detail for each is below.

- [ ] **1** — Write the check script
- [ ] **2** — Rename the deployment configuration to `boarhq`
- [ ] **3** — `[P]` Publish the Serve handler through Funnel
- [ ] **4** — `[P]` Clear inherited identifiers and the tailnet-only claim from the deployment files
- [ ] **5** — `[P]` Correct `docs/infrastructure.md`
- [ ] **6** — `[P]` Cut the inherited application out of `docs/architecture.md`
- [ ] **7** — `[P]` Correct `docs/compose.md`
- [ ] **8** — `[P]` Remove the origin clause from the spec Problem sections
- [ ] **9** — `[P]` Remove the origin from the index and the progress file
- [ ] **10** — `[P]` Record the exposure in `CLAUDE.md`
- [ ] **11** — Verify from outside — needs the operator

## Tasks

### 1. Write the check script

- **Plan:** Test strategy. **Spec:** FR3, FR5, FR6.
- **Files:** `deploy/check/001-public-access.sh` (new).
- **Depends on:** nothing.
- **Do:** A POSIX shell script needing only `docker compose`, `jq`, `grep` and
  `git`. Runs the four checks above, reports every failing check rather than
  stopping at the first, names what it found (service, path, or file and
  line), and exits non-zero if any failed. **ports** resolves the
  configuration with `docker compose config --format json`, supplying
  placeholder values for the required variables so no `.env` is read. **paths**
  and **names** work from `git ls-files`.
- **Acceptance:**
  - *FR3 — Given the resolved deployment configuration, when it is inspected,
    then no service publishes a port on the host.* **ports** passes on the
    repository as it is, and fails — naming the service — when run against a
    copy of `compose.yaml` with a `ports:` entry added to one service. The copy
    lives in the scratch directory, not the repository.
  - **paths** fails today, naming `deploy/tailscale/cdm.json`.
  - **names** fails today, listing matches in fourteen files.
  - **ids** fails today, listing 33 lines.
  - The script's own text does not trip **names**: it spells its search terms
    so that it does not match itself.

### 2. Rename the deployment configuration to `boarhq`

- **Plan:** Architecture §2, Contracts (External URL). **Spec:** FR1, FR5.
- **Files:** `compose.yaml`; `deploy/tailscale/cdm.json` →
  `deploy/tailscale/boarhq.json` (`git mv`, content unchanged); `.env.example`.
- **Depends on:** 1.
- **Do:** Apply every row of the rename table in plan §2 that falls in these
  files, to the exact values in the Contracts table: compose project, sidecar
  service and its three references, hostname, state volume, Serve
  configuration path and mount, tag, the five external-URL settings, realm,
  client id, the five `BOARHQ_*` variables, database and role including the
  health check and the DSN, and the title in the header comments. In
  `.env.example` also the tag and client-id mentions. One commit.
- **Not here:** the `R<n>`/`D<n>` identifiers and the "Reachable ONLY from the
  tailnet" header in `compose.yaml` (task 4); the Funnel block (task 3).
- **Note — needs a decision before it is touched:** `.env.example` says the
  forward-auth gateway is "only used by FUTURE services" and that "CDM's own
  web and api authenticate directly". Removing the name leaves a sentence that
  is still false; `specs/README.md` already records it as a stale comment.
  This task removes the name only. If the sentence should be corrected here
  instead, say so before the task runs.
- **Acceptance:**
  - *FR5 — Given the repository's files, when they are searched for `cdm` …
    then the only matches are inside `specs/001-public-access/`.* For this
    task's files: no match in `compose.yaml`, `.env.example` or
    `deploy/tailscale/`.
  - **paths** passes.
  - **ports** still passes, which also proves the configuration still
    resolves.
  - The resolved configuration shows the eight values of the Contracts table
    exactly.

### 3. `[P]` Publish the Serve handler through Funnel

- **Plan:** Architecture §1 and "Outside the repository", Contracts (Serve
  configuration). **Spec:** FR1, FR2, FR4.
- **Files:** `deploy/tailscale/boarhq.json`; `.env.example`.
- **Depends on:** 2.
- **Do:** Make `boarhq.json` the complete file given in Contracts — the
  existing content plus the `AllowFunnel` block. In `.env.example`, extend the
  auth-key note: the key's tag must be granted the `funnel` attribute in the
  tailnet policy.
- **Acceptance:**
  - `boarhq.json` parses as JSON and equals the Contracts file.
  - **ports** still passes (FR3 is what keeps Funnel from needing a host
    port).
  - FR1, FR2 and FR4 cannot be checked here; they are task 11.

### 4. `[P]` Clear inherited identifiers and the tailnet-only claim from the deployment files

- **Plan:** Architecture §2b (identifiers row), §3 (`compose.yaml` row).
  **Spec:** FR6.
- **Files:** `compose.yaml` (comments only); `deploy/caddy/Caddyfile` (comments
  only).
- **Depends on:** 2.
- **Do:** Delete each `R<n>`/`D<n>` identifier and keep the reasoning beside
  it. Rewrite the `compose.yaml` header's "Reachable ONLY from the tailnet"
  and its diagram line to state public reach through Funnel and why. In the
  Caddyfile, replace the remaining old names in comments (title, sidecar
  service name, Serve configuration file name). No directive, value or
  ordering changes in either file.
- **Acceptance:**
  - *FR6 — Given the deployment files and the documents, when they are read,
    then no comment or sentence cites a requirement or decision identifier of
    the inherited project.* **ids** reports nothing in `compose.yaml` or
    `deploy/`.
  - No match for `cdm` in `deploy/`.
  - `git diff` for both files, with comment lines filtered out, is empty.

### 5. `[P]` Correct `docs/infrastructure.md`

- **Plan:** Architecture §2 (documents), §2b, §3, Go-live rule, "Outside the
  repository". **Spec:** FR1, FR5, FR6, the SEC1 non-functional requirement.
- **Files:** `docs/infrastructure.md`.
- **Depends on:** 1.
- **Do:** Title and intro no longer name the origin, no longer say "never
  exposed to the public internet" or "everything is still named `cdm`", and no
  longer list the inherited identifiers. Names follow the rename table. §
  Commands gains the four operator prerequisites and the go-live rule with its
  command. § Known exposures is replaced by a pointer to the binding in
  `CLAUDE.md`; the "enabling Funnel requires a decision" sentence goes. The
  pointer to `docs/architecture.md` sections follows task 6's renumbering:
  §1, §2 and the sign-in sequence, now §3.
- **Acceptance:**
  - **names** and **ids** report nothing in this file.
  - The file states the go-live rule: the identity provider is not started on
    a Funnel-enabled node until 003 and 004 are validated.
  - Nothing in the file says the stack is private or that there are no
    exposures.

### 6. `[P]` Cut the inherited application out of `docs/architecture.md`

- **Plan:** Architecture §2b (application design, link, dead pointers,
  identifiers), §2 (documents), §3. **Spec:** FR5, FR6.
- **Files:** `docs/architecture.md`.
- **Depends on:** 1.
- **Do:** Delete §3 Domain model, §4 Cost category matrix, §6 Re-check
  sequence, §7 Offer lifecycle, and the depreciation and scheduler items of §8.
  Renumber: Sign-in sequence becomes §3, Open architectural decisions §4.
  Remove the "robots gate" and offer-source nodes and their edges from the §1
  diagram. Delete the rendered-version link and the `REQUIREMENTS.md` and
  `input/_analysis.md` pointers. Delete the identifiers, keeping the reasoning.
  Apply the rename, including the diagram node id. Correct the three
  statements Funnel makes false: the "no public ingress" label, "Funnel … is
  not used", "Tailnet membership is the outer gate".
- **Acceptance:**
  - *FR6 — … and no section describes its application.* The file's headings
    are: Deployment topology, Tailscale, Sign-in sequence, Open architectural
    decisions.
  - **names** and **ids** report nothing in this file.
  - Both remaining Mermaid diagrams still parse — every node referenced by an
    edge is defined.
  - This is a deletion of roughly half the file; say so in the commit body.
    The text stays recoverable from git.

### 7. `[P]` Correct `docs/compose.md`

- **Plan:** Architecture §2 (documents), §2b (leaks, identifiers), §3.
  **Spec:** FR5, FR6.
- **Files:** `docs/compose.md`.
- **Depends on:** 1.
- **Do:** Apply the rename everywhere the file repeats a name, including
  diagram node ids, the example volume name and the `-1` troubleshooting row.
  Delete the identifiers, keeping the reasoning. Remove the external
  offer-sources node and its egress edge; reword the volume label and the
  reset warning to "application data". Change the "no public ingress" diagram
  label, and replace the §10 row on enabling Funnel with one saying it is
  enabled and pointing at the Known-exposures binding.
- **Not here:** the §10 row about a service that cannot run under a path. It
  belongs to `002-single-hostname-routing`.
- **Acceptance:**
  - **names** and **ids** report nothing in this file.
  - Every Mermaid diagram still parses.
  - No sentence says Funnel is off or that the stack has no public ingress.

### 8. `[P]` Remove the origin clause from the spec Problem sections

- **Plan:** Architecture §2b (origin sentences). **Spec:** FR6.
- **Files:** `specs/002-single-hostname-routing/spec.md`,
  `specs/003-authentication-gate/spec.md`,
  `specs/004-delegated-identity/spec.md`, `specs/005-persistence/spec.md`,
  `specs/006-secrets-and-configuration/spec.md`,
  `specs/007-startup-and-recovery/spec.md`, and the Problem section of this
  spec's own `spec.md`. Seven files, one identical one-clause edit each.
- **Depends on:** 1.
- **Do:** Delete the "Inherited from the … infrastructure
  (`docs/infrastructure.md`)" sentence from each Problem section, and the name
  from 006's resolved note. In this spec's Problem section, reword "This
  reverses the inherited … infrastructure, which was private-network-only" to
  say what it reverses without naming a source. Nothing else in any spec
  changes — no requirement, criterion or status.
- **Acceptance:**
  - **names** reports nothing under `specs/002-*` through `specs/007-*`.
  - `git diff --stat` shows one or two changed lines per file.

### 9. `[P]` Remove the origin from the index and the progress file

- **Plan:** Architecture §2 (documents), §2b (origin sentences). **Spec:** FR5,
  FR6.
- **Files:** `specs/README.md`; `rules/_progress.md`.
- **Depends on:** 1.
- **Do:** In `specs/README.md`: § "Where these seven came from" keeps its two
  facts — the specs were written after the files they describe, and none has
  been run — without naming where the files came from; the 001 row of the gap
  table uses the new Serve configuration path; the struck-through finding
  loses the name. In `rules/_progress.md`: the "inherited files no longer
  match the specs" follow-up loses the name and its "everything is still named
  `cdm`" clause, which this feature closes.
- **Acceptance:**
  - **names** reports nothing in either file.
  - No row is added to or removed from any table, and no status changes.

### 10. `[P]` Record the exposure in `CLAUDE.md`

- **Plan:** Architecture §3, Contracts (Known-exposures entry). **Spec:** the
  SEC1 non-functional requirement.
- **Files:** `CLAUDE.md`.
- **Depends on:** nothing.
- **Do:** Replace the placeholder under **Known exposures** in § Bindings with
  the entry given in Contracts, verbatim. No other binding is filled in.
- **Acceptance:**
  - The binding names the hostname, the three unauthenticated route groups,
    the reason accepted, what would remove it, and that the identity provider
    is not yet live under Funnel.
  - `git diff` touches only that binding.

### 11. Verify from outside — needs the operator

- **Plan:** "Outside the repository", Go-live rule, Test strategy (deployed
  rows). **Spec:** FR1, FR2, FR4.
- **Files:** none changed. Results go to `notes.md` at `/sdd-validate`.
- **Depends on:** 2–10 merged, and the script exiting 0.
- **Cannot be done by the agent alone.** It needs the tailnet policy edited, a
  new auth key in `.env` (never shown to the agent — SEC4), and a device
  outside the private network.
- **Do:** The operator completes the four prerequisites, then starts only the
  ingress and the proxy: `docker compose up -d ts-boarhq proxy`. The identity
  provider is not started (go-live rule).
- **Acceptance:**
  - *FR1 — Given a device that is not a member of the operator's private
    network, when it requests the stack's hostname, then it receives a
    response.* From a phone on mobile data,
    `curl -sS -o /dev/null -w '%{http_code}' https://boarhq.<tailnet>.ts.net/`
    prints an HTTP status. 502 is the expected one.
  - *FR2 — Given such a device with no manually installed certificate, when it
    opens the stack's hostname in a browser, then the page loads over HTTPS
    with no certificate warning.*
  - *FR4 — Given a device on the internet, when it requests the host machine's
    own address on any port, then no service of the stack responds.*
    `nmap -Pn -p- <host's public address>` from outside shows no port of this
    stack.
  - The node registered as `boarhq`, not `boarhq-1`.
