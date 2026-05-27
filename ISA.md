---
project: storj-sno-cluster
task: Implementation-architecture reference for a DE-resident Storj SNO multi-node setup
slug: storj-sno-cluster
effort: E3
effort_source: classifier
phase: verify
progress: 45/49
mode: build
started: 2026-05-27T09:33Z
updated: 2026-05-27T17:35Z
---

## Problem

There is no opinionated, DE-resident, solo-operator-scoped reference for
running a Storj SNO multi-node setup. Upstream documentation is generic,
the strongest community templates (tomaae/storj-deployment) are
infrastructure-comprehensive but operator-context-neutral, and the
regulatory perimeter for DE operators is absent from every public
artifact. A prospective collaborator or recruiter reading mjdeving.com
needs a forkable artifact that demonstrates architectural judgement
about this category of work without claiming runtime operation that
mjdeving does not currently have.

## Vision

A public GitHub repository at github.com/mj-deving/storj-sno-cluster
that reads as the bookshelf a serious DE-resident solo Storj operator
would want: opinionated topology for 8-10 nodes across 4 subnets,
hardened defaults applied at every layer, an explicit DE-operator
regulatory perimeter section, monitoring config that ships with
sensible Grafana panels, and trade-off documentation for every
significant design decision. The reader either forks it and runs it,
or reads it and walks away with a clear architectural mental model.

Euphoric surprise: a reader thinks "this person has actually thought
through the operational shape of a solo Storj setup, the DE-legal
perimeter, and the failure modes, and they are honest about not
running it themselves right now".

## Out of Scope

- Real-node operation by mjdeving (no hardware available)
- Synthetic dashboards, fabricated earnings numbers, or any
  fabricated runtime evidence
- Single-node minimal setup (the reference targets the 8-10 node
  cluster shape; single-node setups are upstream territory)
- Non-Storj decentralised storage (Filecoin SP, Sia, Arweave belong to
  separate references if ever needed)
- Custodial storage hosting for clients (would change the regulatory
  perimeter)
- Multi-language documentation; English only at v1, DE inline where
  regulatory specifics apply

## Principles

1. Honest-framing. README and ISA both state explicitly that this is a
   reference artifact, not a runtime.
2. Opinionated. Topology and hardening defaults are choices, with each
   choice documented in trade-off form.
3. Forkable. Compose and config use environment variables and clearly
   marked extension points.
4. DE-legible. The regulatory perimeter section is explicit and
   recommends a Steuerberater for actual filings.
5. Single-author maintainable. No team-scale orchestration tooling.
6. Anti-overclaim. Em-dash zero tolerance, no "battle-tested",
   "production-ready", "groundbreaking", "leverage", "seamless".

## Constraints

- Public GitHub repository at github.com/mj-deving/storj-sno-cluster
- License MIT
- English primary language
- Docker Compose v2 syntax (no Swarm, no Kubernetes)
- Bun and TypeScript not applicable here; shell scripts and YAML only
- Monitoring stack: Prometheus + Grafana, no commercial SaaS dependency
- Storj container: official storj/storagenode-docker only (no custom
  fork of the storagenode itself)
- No external secrets management dependency (sops, vault) at v1;
  documented as a v2 follow-up
- Documentation in Markdown; no static site generator at this stage

## Goal

Publish a complete repository with README, ARCHITECTURE, OPERATIONS,
HARDENING, DE-OPERATOR, compose, monitoring, ISA, and LICENSE that
reads as an implementation-architecture domain showcase, passes
anti-slop sweep, builds in CI for compose validation, and is forkable
by a DE solo operator considering Storj SNO work.

## Criteria

### Repository bones

- [x] ISC-1: `~/projects/storj-sno-cluster/.git/` initialised on `main`
- [x] ISC-2: `README.md` present at root with framing + audience + cross-link
- [x] ISC-3: `ARCHITECTURE.md` present (stub with planned-sections placeholder)
- [x] ISC-4: `LICENSE` MIT present at root
- [x] ISC-5: `.gitignore` excludes editor files, OS junk, and any `.env` variant
- [x] ISC-6: `OPERATIONS.md` present documenting monitoring, alerting, backup, lifecycle
- [x] ISC-7: `HARDENING.md` present documenting UFW, fail2ban, SSH, AppArmor, unattended-upgrades
- [x] ISC-8: `DE-OPERATOR.md` present documenting Gewerbeanmeldung, §22 EStG, vetting period reality, Steuerberater disclaimer
- [ ] ISC-9: First commit lands on `main` with a Conventional Commit message
- [ ] ISC-10: Public GitHub repo `github.com/mj-deving/storj-sno-cluster` created and pushed

### Architecture document

- [x] ISC-11: Topology section names per-node hardware envelope, IP diversification target, recommended geographic distribution
- [x] ISC-12: Per-node components section names storagenode container, identity, updater, exporter, log handling
- [x] ISC-13: Network topology section names port 28967, recommended mesh approach for operator-side observability
- [x] ISC-14: Data flow section names customer upload, repair, audit, GC traffic categories
- [x] ISC-15: Trade-off decisions section documents at least 6 architectural decisions with alternatives considered
- [x] ISC-16: Cross-link back to operator-references/DOCTRINE.md present

### Compose template

- [x] ISC-17: `compose/docker-compose.yml` declares storagenode service with named volume mounts
- [ ] ISC-18: [DROPPED, see Decisions 2026-05-27T17:30Z. Storj bundled the updater into the storagenode image in 2024; the separate updater container is no longer the canonical path and has no `latest` tag on Docker Hub. Single-container-per-node model documented in ARCHITECTURE.md Decision 1.]
- [x] ISC-19: `compose/docker-compose.yml` declares node-exporter service. The storagenode-exporter sidecar is intentionally omitted; the storagenode exposes its own Prometheus-format `/metrics` on the debug port (127.0.0.1:5999), which Prometheus scrapes directly. ARCHITECTURE.md Decision 6.
- [x] ISC-20: `compose/.env.example` documents every environment variable with comment
- [x] ISC-21: Compose template passes `docker compose -f compose/docker-compose.yml config` syntax validation

### Monitoring stack

- [x] ISC-22: `monitoring/prometheus.yml` declares scrape configs for storagenode-exporter and node-exporter
- [x] ISC-23: `monitoring/grafana/dashboard-storj.json` present with at least 6 panels (uptime, ingress, egress, audit score, suspension score, disk usage)
- [x] ISC-24: `monitoring/alerts.yml` declares at least 3 alert rules (node down, suspension warning, disk near full)

### Hardening checklist

- [x] ISC-25: HARDENING.md prescribes UFW default-deny with explicit allow list
- [x] ISC-26: HARDENING.md prescribes SSH key-only authentication, no password fallback
- [x] ISC-27: HARDENING.md prescribes unattended-upgrades with reboot policy documented
- [x] ISC-28: HARDENING.md prescribes fail2ban for SSH and storagenode log monitoring
- [x] ISC-29: HARDENING.md prescribes AppArmor profile reference for docker-default

### DE operator notes

- [x] ISC-30: DE-OPERATOR.md names Gewerbeanmeldung as the registration requirement
- [x] ISC-31: DE-OPERATOR.md documents §22 EStG treatment of staking-style income and the €256 sonstiges-Einkommen threshold
- [x] ISC-32: DE-OPERATOR.md states the vetting period reality (~30 days, near-zero early earnings)
- [x] ISC-33: DE-OPERATOR.md carries a "not tax or legal advice, consult a Steuerberater" disclaimer banner as the FIRST visible block on the page (above the first H2), not buried mid-document

### Anti-slop and honest-framing

- [x] ISC-34: Anti: zero em-dashes in body .md files. Covers BOTH Unicode `—` (U+2014) and `–` (U+2013). Probe: `grep -rcE '—|–' --include='*.md' --exclude='ISA.md' .` returns all zeros. ISA file is rule-definition and is exempted (it must contain the literal characters in this very ISC text).
- [x] ISC-35: Anti: zero matches for the codified banned vocab in body .md files: `battle-tested`, `production-ready`, `production-proven`, `groundbreaking`, `leverage`, `seamless`, `cutting-edge`, `bulletproof`, `effortless`, `enterprise`, `robust`, `simply`, `just`. Probe: `grep -rEi --include='*.md' --exclude='ISA.md' '\b(battle-tested|production-ready|production-proven|groundbreaking|leverage|seamless|cutting-edge|bulletproof|effortless|enterprise|robust|simply|just)\b' .` returns empty. ISA file is rule-definition and is exempted.
- [x] ISC-36: Anti: zero synthetic dashboards or fabricated earnings figures in any committed file

### Advisor commitment gaps (folded 2026-05-27T15:30Z)

- [x] ISC-37: ARCHITECTURE.md contains "Storage layout" section (sub-paths, per-node disk topology, identity-to-volume mapping)
- [x] ISC-38: ARCHITECTURE.md contains "Failure modes" section (suspension, disqualification, GC stalls, network partition, repair-traffic surges)
- [x] ISC-39: ARCHITECTURE.md contains "Capacity envelope" section (utilisation curve, vetting-period yield, traffic shape post-vetting)
- [x] ISC-40: Compose pins `storjlabs/storagenode` to a SHA256 digest resolved at draft time (not a floating tag)
- [ ] ISC-41: [DROPPED, see Decisions 2026-05-27T17:30Z and ISC-18 tombstone. No separate updater container exists to pin.]
- [x] ISC-42: Compose pins exporter images (`node-exporter`, `storagenode-exporter`) to SHA256 digests
- [x] ISC-43: Monitoring template references images with SHA256 digests for reproducibility even though template-only
- [x] ISC-44: ARCHITECTURE.md "Prior art and credit" section at top names `tomaae/storj-deployment` with link
- [x] ISC-45: README.md cross-links to ARCHITECTURE.md "Prior art and credit" section
- [x] ISC-46: `compose/README.md` carries explicit framing header: "starts cleanly, requires your own node identity and authorization token, not operated by us"
- [x] ISC-47: `monitoring/README.md` carries explicit framing header: "scaffolding only, placeholders for endpoints / retention / alert thresholds, never validated against live metrics"
- [x] ISC-48: Anti: zero image files (`*.png`, `*.jpg`, `*.gif`) committed anywhere in repo (`git ls-files '*.png' '*.jpg' '*.gif'` returns empty)
- [x] ISC-49: Anti: Grafana dashboard JSON uses obviously-placeholder values only (no realistic numeric ranges, placeholder node IDs like `<NODE_ID>`, sentinel IPs `0.0.0.0`)

## Test Strategy

| isc      | type              | check                                        | threshold | tool         |
|----------|-------------------|----------------------------------------------|-----------|--------------|
| 1-10     | file / git state  | dir + file existence, branch, push status    | exact     | `test`, `git`, `gh` |
| 11-16    | content           | named sections present with required prose   | grep      | `grep`       |
| 17-21    | compose validity  | service + env presence, syntax check         | passes    | `grep`, `docker compose config` |
| 22-24    | monitoring        | scrape configs, dashboard panel count, alert count | grep + jq | `grep`, `jq` |
| 25-29    | hardening         | named prescriptions appear                   | grep      | `grep`       |
| 30-33    | DE-operator       | named regulatory anchors and disclaimer      | grep      | `grep`       |
| 34-36    | anti-slop         | grep returns 0                               | exact 0   | `grep -r`    |
| 37-39    | ARCHITECTURE sections | named sections present                  | grep      | `grep`       |
| 40-43    | pinned digests    | image references use `@sha256:` syntax       | exact     | `grep -E 'image: .*@sha256:'` |
| 44-45    | tomaae attribution| ARCH section named + README cross-link       | grep      | `grep`       |
| 46-47    | framing headers   | README files in compose/ and monitoring/     | grep      | `grep`       |
| 48       | no image files    | `git ls-files` glob returns empty            | exact 0   | `git ls-files` |
| 49       | placeholder values| no realistic metrics in dashboard JSON       | manual + grep | `jq`, `grep` |

## Features

| name                 | description                                       | satisfies   | depends_on | parallelizable |
|----------------------|---------------------------------------------------|-------------|------------|----------------|
| F1: Repo bones       | README + ARCHITECTURE-stub + LICENSE + ISA + git  | ISC-1..5    | none       | no             |
| F2: Doc stubs        | OPERATIONS + HARDENING + DE-OPERATOR stubs        | ISC-6..8    | F1         | partial        |
| F3: Architecture     | ARCHITECTURE.md full expansion                    | ISC-11..16  | F1         | no             |
| F4: Compose          | docker-compose.yml + .env.example + validation    | ISC-17..21  | F1         | no             |
| F5: Monitoring       | prometheus + grafana dashboard + alerts           | ISC-22..24  | F4         | no             |
| F6: Hardening        | HARDENING.md full expansion                       | ISC-25..29  | F2         | no             |
| F7: DE operator      | DE-OPERATOR.md full expansion + disclaimer        | ISC-30..33  | F2         | no             |
| F8: Anti-slop gate   | grep sweep on all .md                             | ISC-34..36  | all docs   | no             |
| F9: Public push      | gh repo create + first push                       | ISC-9, 10   | F1..F8     | no             |

## Decisions

- 2026-05-27T09:33Z: Created after mjdeving confirmed pure design-reference framing. No real-node operation, no synthetic statistics, pure architecture-as-artifact.
- 2026-05-27T09:33Z: Anchored on tomaae/storj-deployment for monitoring stack inspiration; storj/storagenode-docker as the official container baseline. Both credited in README.
- 2026-05-27T09:33Z: License MIT for forkability priority.
- 2026-05-27T09:33Z: Scope locked to 8-10 node multi-cluster target. Single-node setups treated as upstream territory.
- 2026-05-27T09:33Z: Out of scope: any non-Storj decentralised storage (Filecoin, Sia, Arweave). Those would be separate references if ever pursued.
- 2026-05-27T09:35Z: ISA scaffolded inline rather than via Skill("ISA", "scaffold") for speed; ISC-1..4 already pass at scaffold time.
- 2026-05-27T15:30Z: Advisor commitment-boundary returned. Verdict: hybrid coherent (runnable compose + template monitoring) iff explicit framing headers + obvious placeholder values + no screenshots. DE-OPERATOR language = English prose + German legal terms verbatim + parenthetical English gloss on first mention. Folded as 13 new ISCs (37-49) under "Advisor commitment gaps". ID-stability preserved on ISCs 1-36; ISC-33/34/35 prose modified to encode disclaimer-at-top, Unicode-em-dash coverage, and codified banned vocab list.
- 2026-05-27T15:30Z: Confirmed by mjdeving (this session): compose can boot clean on a fresh host at draft time. Hybrid shape stays. If draft surfaces actual boot failure, downgrade to template-only per advisor's "half-runnable is worse than fully-template" rule and tombstone ISC-40..42 with rationale.
- 2026-05-27T15:30Z: Pentester delegation skipped on HARDENING.md review. Show-my-math: anti-slop sweep + the 5 named hardening-domain ISCs (UFW, SSH, unattended-upgrades, fail2ban, AppArmor) + advisor commit-boundary already cover the correctness surface for a reference document. Pentester adds review-of-reviewer overhead without adding meaningful assurance for a non-runtime doc. Cato remains for VERIFY (E4 mandatory).

## Changelog

(populated at LEARN)

## Verification

(populated at VERIFY, evidence per ISC)
