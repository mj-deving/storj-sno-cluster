---
project: storj-sno-cluster
task: Implementation-architecture reference for a DE-resident Storj SNO multi-node setup
slug: storj-sno-cluster
effort: E3
effort_source: classifier
phase: build
progress: 4/36
mode: build
started: 2026-05-27T09:33Z
updated: 2026-05-27T09:35Z
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
Marius does not currently have.

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

- Real-node operation by Marius (no hardware available)
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
- [ ] ISC-5: `.gitignore` excludes editor files, OS junk, and any `.env` variant
- [ ] ISC-6: `OPERATIONS.md` present documenting monitoring, alerting, backup, lifecycle
- [ ] ISC-7: `HARDENING.md` present documenting UFW, fail2ban, SSH, AppArmor, unattended-upgrades
- [ ] ISC-8: `DE-OPERATOR.md` present documenting Gewerbeanmeldung, §22 EStG, vetting period reality, Steuerberater disclaimer
- [ ] ISC-9: First commit lands on `main` with a Conventional Commit message
- [ ] ISC-10: Public GitHub repo `github.com/mj-deving/storj-sno-cluster` created and pushed

### Architecture document

- [ ] ISC-11: Topology section names per-node hardware envelope, IP diversification target, recommended geographic distribution
- [ ] ISC-12: Per-node components section names storagenode container, identity, updater, exporter, log handling
- [ ] ISC-13: Network topology section names port 28967, recommended mesh approach for operator-side observability
- [ ] ISC-14: Data flow section names customer upload, repair, audit, GC traffic categories
- [ ] ISC-15: Trade-off decisions section documents at least 6 architectural decisions with alternatives considered
- [ ] ISC-16: Cross-link back to operator-references/DOCTRINE.md present

### Compose template

- [ ] ISC-17: `compose/docker-compose.yml` declares storagenode service with named volume mounts
- [ ] ISC-18: `compose/docker-compose.yml` declares storagenode-updater service paired with storagenode
- [ ] ISC-19: `compose/docker-compose.yml` declares node-exporter and storagenode-exporter services
- [ ] ISC-20: `compose/.env.example` documents every environment variable with comment
- [ ] ISC-21: Compose template passes `docker compose -f compose/docker-compose.yml config` syntax validation

### Monitoring stack

- [ ] ISC-22: `monitoring/prometheus.yml` declares scrape configs for storagenode-exporter and node-exporter
- [ ] ISC-23: `monitoring/grafana/dashboard-storj.json` present with at least 6 panels (uptime, ingress, egress, audit score, suspension score, disk usage)
- [ ] ISC-24: `monitoring/alerts.yml` declares at least 3 alert rules (node down, suspension warning, disk near full)

### Hardening checklist

- [ ] ISC-25: HARDENING.md prescribes UFW default-deny with explicit allow list
- [ ] ISC-26: HARDENING.md prescribes SSH key-only authentication, no password fallback
- [ ] ISC-27: HARDENING.md prescribes unattended-upgrades with reboot policy documented
- [ ] ISC-28: HARDENING.md prescribes fail2ban for SSH and storagenode log monitoring
- [ ] ISC-29: HARDENING.md prescribes AppArmor profile reference for docker-default

### DE operator notes

- [ ] ISC-30: DE-OPERATOR.md names Gewerbeanmeldung as the registration requirement
- [ ] ISC-31: DE-OPERATOR.md documents §22 EStG treatment of staking-style income and the €256 sonstiges-Einkommen threshold
- [ ] ISC-32: DE-OPERATOR.md states the vetting period reality (~30 days, near-zero early earnings)
- [ ] ISC-33: DE-OPERATOR.md carries a "consult a Steuerberater" disclaimer prominently

### Anti-slop and honest-framing

- [ ] ISC-34: Anti: zero em-dashes in any .md file (`grep -r '—' --include='*.md' .` returns 0)
- [ ] ISC-35: Anti: zero matches for "battle-tested", "production-ready", "groundbreaking", "leverage", "seamless", "cutting-edge" in any .md
- [ ] ISC-36: Anti: zero synthetic dashboards or fabricated earnings figures in any committed file

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

- 2026-05-27T09:33Z: Created after Marius confirmed pure design-reference framing. No real-node operation, no synthetic statistics, pure architecture-as-artifact.
- 2026-05-27T09:33Z: Anchored on tomaae/storj-deployment for monitoring stack inspiration; storj/storagenode-docker as the official container baseline. Both credited in README.
- 2026-05-27T09:33Z: License MIT for forkability priority.
- 2026-05-27T09:33Z: Scope locked to 8-10 node multi-cluster target. Single-node setups treated as upstream territory.
- 2026-05-27T09:33Z: Out of scope: any non-Storj decentralised storage (Filecoin, Sia, Arweave). Those would be separate references if ever pursued.
- 2026-05-27T09:35Z: ISA scaffolded inline rather than via Skill("ISA", "scaffold") for speed; ISC-1..4 already pass at scaffold time.

## Changelog

(populated at LEARN)

## Verification

(populated at VERIFY, evidence per ISC)
