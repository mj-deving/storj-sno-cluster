# storj-sno-cluster

An implementation-architecture reference for a multi-node Storj Storage
Node operator setup run by a single operator.

It documents the topology, the components, the data flow, the trade-offs,
and the hardening defaults for a 1-to-10 node Storj SNO cluster.

## What this is

- A reusable Docker Compose topology for a 1-to-10 node Storj SNO setup
- Prometheus + Grafana monitoring stack configuration
- A host hardening profile (UFW, fail2ban, SSH, AppArmor, unattended-upgrades)

## Target setup shape

- 8-10 HDDs across 4 distinct /24 subnets (home, second home, colo,
  family member, regional VPS with attached storage)
- 2-4 TB per node target
- Stable internet with port-forwarding (default Storj port 28967)
- Hands-on with Linux, Docker, basic networking

The template is filesystem- and host-agnostic; the operational framing is
calibrated for that cluster shape but the topology reads independently of it.

## Repository structure

```
storj-sno-cluster/
├── README.md          this file
├── ARCHITECTURE.md    topology, components, data flow
├── OPERATIONS.md      monitoring, alerting, backup, lifecycle
├── HARDENING.md       host security baseline applied at every layer
├── compose/           Docker Compose template and example env
├── monitoring/        Prometheus + Grafana config and dashboards
└── LICENSE            MIT
```

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

[mjdeving](https://mjdeving.com).

## License

MIT. See [LICENSE](./LICENSE).
