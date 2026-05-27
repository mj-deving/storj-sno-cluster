# storj-sno-cluster

An implementation-architecture reference for a multi-node Storj Storage
Node operator setup, designed for DE-resident solo operators.

This repository is a reference artifact, not a production deployment.
It documents the architecture, the trade-offs, the regulatory boundary,
and the hardening defaults you would want if you were going to build
this. No runtime statistics, no earnings claims.

Part of the [operator-references](https://github.com/mj-deving/operator-references)
collection.

## What this is

- A reusable Docker Compose topology for a 1-to-10 node Storj SNO setup
- Prometheus + Grafana monitoring stack configuration
- Hardening checklist (UFW, fail2ban, SSH, AppArmor, unattended-upgrades)
- DE-operator notes on Gewerbeanmeldung, §22 EStG taxation, vetting reality
- Cross-architecture context via [operator-references/DOCTRINE.md](https://github.com/mj-deving/operator-references/blob/main/DOCTRINE.md)

## What this is not

- A turnkey runtime. You bring your own hardware, IPs, and capital.
- A guarantee of vetting outcome or earnings.
- A template that has been operated under sustained customer load.
  It has not. The architecture is the deliverable, not a runtime
  history.
- Legal or tax advice. Consult a Steuerberater for actual filings.

## Target operator profile

- DE-resident, registered Gewerbe (Einzelunternehmen acceptable)
- 8-10 HDDs available across 4 distinct /24 subnets (home, second home,
  colo, family member, regional VPS with attached storage)
- 2-4 TB per node target (smaller nodes work; smaller nodes earn less)
- Stable internet with port-forwarding (default Storj port 28967)
- Hands-on with Linux, Docker, basic networking

If you do not match this profile, the template is still readable, but
the operational and regulatory framing is calibrated for that shape.

## Repository structure

```
storj-sno-cluster/
├── README.md          this file
├── ARCHITECTURE.md    topology, components, data flow
├── OPERATIONS.md      monitoring, alerting, backup, lifecycle
├── HARDENING.md       security baseline applied at every layer
├── DE-OPERATOR.md     regulatory perimeter, tax notes, disclaimers
├── compose/           Docker Compose template and example env
├── monitoring/        Prometheus + Grafana config and dashboards
├── ISA.md             Ideal State Articulation (project specification)
└── LICENSE            MIT
```

Several of these files are stubs in the initial scaffold; expansion lands
in subsequent commits, gated by ISC-tracked criteria in `ISA.md`.

## Prior art and credit

The canonical attribution lives in [`ARCHITECTURE.md` § Prior art and
credit](./ARCHITECTURE.md#prior-art-and-credit) with pinned upstream
versions. Summary:

- [storj/storagenode-docker](https://github.com/storj/storagenode-docker)
  is the auto-updating official container.
- [tomaae/storj-deployment](https://github.com/tomaae/storj-deployment)
  is the most comprehensive community multi-node Compose reference; the
  monitoring stack here draws from that work.
- [docs.storj.io/node](https://docs.storj.io/node) for operator-facing
  documentation.

This repository is opinionated where the upstream is generic, and
constrained where the upstream leaves the operator to choose.

## Author

[mjdeving](https://mjdeving.com). Dipl.-Ing. Elektrotechnik
(TU Dresden, 2018). Self-employed since 2022 in cloud infrastructure,
now building AI agents and automations.

## License

MIT. See [LICENSE](./LICENSE).
