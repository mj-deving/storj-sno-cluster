# Architecture

Topology, components, and data flow for the storj-sno-cluster reference.

> This document is a stub at v0.1.0. Expansion is tracked by ISC-5..12 in
> [ISA.md](./ISA.md).

## Planned sections

- **Topology.** Per-node hardware envelope, IP diversification target,
  recommended geographic distribution. The "8-10 nodes across 4 /24
  subnets" target and why each constraint exists.
- **Per-node components.** storagenode container, identity store,
  storagenode-updater sidecar, log collector, exporter, reverse proxy if
  applicable.
- **Network topology.** Port 28967 inbound TCP+UDP, optional Tailscale or
  WireGuard mesh for operator-side observability, monitoring fan-in to
  central Prometheus.
- **Data flow.** Customer-upload through satellite-egress, repair traffic,
  audit traffic, GC, and what each costs the operator.
- **Monitoring topology.** Per-node exporter, central Prometheus,
  Grafana, alert routing.
- **Backup topology.** Identity backups (small, critical) versus piece
  storage (not backed up; resilience is at the network layer).
- **Trade-off decisions.** For each significant decision: alternatives
  considered, why this one, when to revisit.

## Why this topology

(stub, expanded in v0.2.0; rationale ties back to the regulatory and
cashflow envelope documented in [operator-references/DOCTRINE.md](https://github.com/mj-deving/operator-references/blob/main/DOCTRINE.md))
