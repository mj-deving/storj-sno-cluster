# ARCHITECTURE

Implementation-architecture reference for an 8-to-10-node Storj Storage
Node Operator (SNO) cluster operated by a single DE-resident operator.
The document is opinionated. Every significant choice is followed by a
named alternative and a one-line reason.

> **Status:** Reference architecture. Not operated as published. The
> compose template (`compose/`) is intended to start cleanly on a fresh
> host; the monitoring stack (`monitoring/`) is template-only with
> placeholder values that need operator-specific resolution before they
> emit useful signal.

## Prior art and credit

This template stands on three pieces of upstream work and a research
vault on DePIN operator economics.

| Upstream                                                                 | Role                                                       | Pinned at draft |
|--------------------------------------------------------------------------|------------------------------------------------------------|-----------------|
| [storjlabs/storagenode](https://hub.docker.com/r/storjlabs/storagenode)  | Official storagenode container (with built-in auto-update) | `sha256:21ca0ac819d1d66240830d28097eeb693923e51383aa3bb07bf823f9fad1f6e9` |
| [tomaae/storj-deployment](https://github.com/tomaae/storj-deployment)    | Reference for multi-node compose patterns + monitoring     | inspiration, not vendored |
| [docs.storj.io/node](https://docs.storj.io/node)                         | Authoritative operator documentation                       | accessed 2026-05-27 |

The storj/storagenode-updater container is intentionally omitted: as of
2024 the updater is bundled into the storagenode image and runs in-band.

A DePIN-operator decision-surface research vault (storj, akash, lido CSM,
filecoin SP, helium, bittensor) lives in the operator's private notes
and informs the choice to scope this artifact to the 8-to-10-node Storj
case rather than the broader DePIN field. The companion umbrella
repository [operator-references](https://github.com/mj-deving/operator-references)
carries the cross-architecture comparison.

## 1. Topology

The reference targets 8 to 10 nodes operated by a single person, spread
across 4 distinct /24 subnets, in 2 to 4 geographic locations.

### Per-node hardware envelope

| Resource | Floor                        | Recommended                  | Ceiling justification     |
|----------|------------------------------|------------------------------|---------------------------|
| Disk     | 2 TB usable                  | 4 to 8 TB usable             | Beyond 16 TB per node, vetting amortisation gets too slow vs replacement cost |
| RAM      | 1 GB free                    | 2 GB free                    | Storj's working set is modest; surplus RAM is wasted |
| CPU      | 1 modern core                | 1 to 2 modern cores          | Background-priority workload; over-provisioning helps only at peak repair |
| Network  | 100 Mbit symmetric           | 250 Mbit+ symmetric          | Asymmetric uplinks throttle egress (the earning side) |
| Identity | 1 per node (irreplaceable)   | 1 per node, encrypted backup | See OPERATIONS.md §3       |

### IP and subnet diversification

Storj's payout model rewards diversity. Multiple nodes behind the same
/24 subnet count as one node for satellite-side traffic allocation. The
operator therefore wants nodes spread across distinct subnets.

Reference cluster layout:

- 2 to 3 nodes on the operator's primary home connection (one /24)
- 2 to 3 nodes on a second home or family member's connection (second /24)
- 2 to 3 nodes on a small VPS or colo provider (third /24)
- 1 to 2 nodes on a fourth location (fourth /24), optional

Decision: 4 distinct /24 subnets is the practical sweet spot for a solo
operator. Three is workable; two is over-concentrated; five or more
crosses into infrastructure-as-second-job territory.

### Geographic distribution

Geographic diversity matters less to Storj's payout model than IP
diversity, but matters more to operational resilience. A single power
outage or ISP incident can take down all nodes at one location. The
reference targets 2 to 4 geographic locations.

## 2. Identity lifecycle

Each node has exactly one identity, generated once, used forever.

### Generation and authorization

```
1. operator runs `identity create storagenode` on a host with the storj CLI
2. operator runs `identity authorize storagenode <auth-token>` to bind
   the identity to a specific Storj satellite authorization
3. identity files land in ~/.local/share/storj/identity/storagenode/
4. operator copies the identity directory to the per-node host
5. compose mounts the identity directory into the storagenode container
```

The authorization token comes from Storj's signup process. Each
authorization token is one-time-use and binds to one identity. Lose the
identity files and the authorization is gone with them.

### Identity files (per node)

```
identity/
├── ca.cert          (certificate authority cert)
├── ca.key           (CA private key)
├── identity.cert    (node identity cert, signed by CA)
└── identity.key     (node identity private key)
```

Mode `0600` on files, `0700` on the directory, owned by the operator
user that runs the storagenode container.

### Container lifecycle

The storagenode container bundles three concerns into one process:

| Concern         | Mechanism                                                    |
|-----------------|--------------------------------------------------------------|
| Storage daemon  | `storagenode` binary listens on port 28967 (TCP + UDP)       |
| Auto-update     | Built-in updater polls Storj's release feed, pulls new binary, restarts in place |
| Dashboard       | Storagenode dashboard on `127.0.0.1:14002` (loopback only by default) |
| Debug metrics   | Storagenode debug server on `127.0.0.1:5999` (loopback only) |

Pre-bundled updater is the model since 2024. The compose template
therefore declares one storagenode service per node and no separate
updater service. The trade-off section below documents this decision.

### Log handling

Storj logs to stdout by default; the compose template uses Docker's
`json-file` driver with rotation (`max-size: 100m`, `max-file: 5`).

Operators who want long-retention logs should mount a logging-driver
override (`docker-compose.override.yml`) targeting `loki`, `journald`,
or `fluentd` per their existing observability stack.

## 3. Storage layout

The storagenode container writes piece data, metadata SQLite databases,
and configuration to a single mounted directory.

### Per-node directory tree (host side)

```
/opt/storj/
├── identity-1/        (immutable per-node identity, 4 files, mode 0600)
├── identity-2/
├── ...
├── storage-1/         (per-node data, grows to allocation ceiling)
│   ├── config.yaml    (storagenode config, generated on first run)
│   ├── storage/       (piece data, the bulk of the volume)
│   │   ├── blob/
│   │   ├── temp/
│   │   └── trash/     (deleted pieces, GC reclaims this)
│   ├── retain/        (retain process metadata)
│   ├── orders/        (uploaded-piece order records)
│   ├── revocations/   (revocation database)
│   └── *.db           (SQLite piece metadata)
├── storage-2/
└── ...
```

### Identity-to-volume mapping

One node = one identity directory + one storage directory. They are
strictly paired. Mounting identity-1 against storage-2's data would
corrupt both nodes simultaneously.

The compose template encodes this pairing in named environment variables
(`IDENTITY_DIR_1` and `STORAGE_DIR_1` always travel together).

### Filesystem choice (advisory)

| Filesystem | Use it if                                                         | Avoid it if                                                  |
|------------|-------------------------------------------------------------------|--------------------------------------------------------------|
| ext4       | Default Linux, no special needs                                   | Multi-disk pools needed                                      |
| XFS        | Single very large volume per node, frequent small file writes     | You expect to need online shrink                             |
| ZFS        | Multi-disk pool with checksumming and snapshots desired           | Memory is tight (ZFS likes 1 GB RAM per TB of pool)          |
| Btrfs      | Snapshots wanted, single-disk or RAID-1 only                      | RAID-5 / RAID-6 (Btrfs RAID-56 is not stable)                |

Filesystem choice is operator-side. The compose template is filesystem-
agnostic; it mounts whatever directory the operator provides.

### Per-node disk size guidance

- **Floor: 2 TB usable.** Below this, vetting amortisation never reaches
  break-even on hardware cost.
- **Recommended: 4 to 8 TB usable.** The sweet spot. Vetting completes in
  ~30 days; mature node revenue covers electricity + amortised hardware.
- **Ceiling: 16 TB usable.** Beyond this, the operator absorbs
  oversized replacement-risk per node. Larger disks don't scale
  earnings linearly; the Storj payout curve is roughly utilisation-
  proportional.

## 4. Network ingress

The storagenode listens on a single TCP and UDP port (default 28967).
Customer traffic, repair traffic, audit traffic, and dashboard traffic
all converge there.

### Per-node port allocation

For multiple nodes behind one NAT, each node needs a distinct external
port. The convention is to use 28967, 28968, 28969, etc., one per node.
The storagenode `--server.address` flag carries the public address and
port the satellites should advertise.

### Port forwarding posture

| Scenario           | Approach                                                          |
|--------------------|-------------------------------------------------------------------|
| Static public IP   | Forward `28967+N` per node, set `ADDRESS=public.ip:28967+N`       |
| Dynamic IP         | Use a DDNS service, set `ADDRESS=ddns.example.com:28967+N`        |
| IPv6 dual-stack    | Forward both v4 and v6; storj currently routes via v4 in practice |
| CGNAT / no port    | Storj will not work; CGNAT operators need a VPS reverse tunnel    |

CGNAT is a real blocker. Operators behind T-Mobile mobile broadband,
many German fibre providers, and most mobile-tethered setups need to
plan a wireguard tunnel to a public-IP VPS or skip the location.

### Operator-side observability mesh

Monitoring traffic (Prometheus scraping the node-exporter, Grafana
viewing dashboards) MUST NOT traverse public network in cleartext.

Recommended: Wireguard or Tailscale mesh between the operator's
admin host and each node host. Prometheus runs on the admin host;
node-exporter listens on the mesh interface only.

Anti-pattern: exposing node-exporter port 9100 to the public internet
behind no authentication. This is the most common monitoring leak;
node-exporter exposes filesystem and process metadata that becomes a
reconnaissance starting point.

## 5. Failure modes

The failure modes section is the most operationally load-bearing part
of this document. Each failure mode names the trigger, the visible
signal, and the recovery path.

### Suspension

| Trigger     | Audit score drops below the satellite's suspension threshold     |
|-------------|------------------------------------------------------------------|
| Signal      | Per-satellite suspension flag on the storagenode dashboard       |
| Severity    | Recoverable but time-sensitive                                   |
| Recovery    | Fix the root cause (disk error, lost pieces, persistent uptime). The satellite re-vets over a multi-day window. |
| Cliff       | If suspension persists past the satellite's grace window, escalates to disqualification |

### Disqualification

| Trigger     | Audit score falls below the disqualification floor              |
|-------------|------------------------------------------------------------------|
| Signal      | Per-satellite disqualification flag, permanent                  |
| Severity    | Terminal for this node-satellite pair                            |
| Recovery    | None. The identity is dead for this satellite. The node continues to operate on satellites where it remains in good standing; the lost satellite's stored pieces are wasted disk. |
| Implication | An identity disqualified across all satellites is fully dead. Identity files become worthless. |

Anti-pattern: ignoring suspension warnings. The window to recover from
suspension is short; the window after disqualification is zero.

### Garbage collection (GC) stalls

| Trigger     | The retain process cannot keep up with deletion volume; trash accumulates |
|-------------|------------------------------------------------------------------|
| Signal      | Trash directory grows past expected steady-state; disk usage approaches allocation ceiling while pieces-stored count is flat |
| Severity    | Ingress slows or stops; no immediate disqualification risk      |
| Recovery    | Restart the storagenode container; allow the retain process to catch up. Persistent stalls indicate disk I/O saturation or filesystem issues. |

### Network partition

| Partition length | Outcome |
|---|---|
| Short partition (< 5 min)   | Storj's audit timeout is forgiving; no immediate suspension risk |
| Medium partition (5-60 min) | Audit windows may be missed; suspension scoring slides downward  |
| Long partition (> 1 hour)   | Suspension probable on at least one satellite; recovery requires re-vetting once back online |

### Repair traffic surge

| Trigger     | A peer node went offline; satellites re-distribute its pieces to surviving nodes including yours |
|-------------|------------------------------------------------------------------|
| Signal      | Ingress traffic spike above baseline, often 5-to-10x for several hours |
| Severity    | Cosmetic; this is paid traffic, take it                           |
| Operator note | Don't tune alerts to fire on traffic spikes alone; spikes mean repair traffic, which is the network at work |

### Disk failure

| Aspect | Detail |
|---|---|
| Per-node disk failure | One node's data is lost; identity is intact (separate volume) but storage is gone |
| Recovery | The node will be disqualified across all satellites over the next few days as audits fail. No way to recover without the lost pieces. Treat as a node loss. |
| Mitigation | Hardware-RAID under the storage volume reduces this risk but costs disk and complexity. The reference recommends against software RAID for storagenode volumes (storj is already a distributed system; double-redundancy is wasted). |

The per-node identity isolation rationale: keep identity on a separate
volume from storage (often SSD-system-disk for identity, HDD-bulk for
storage). Identity is small (under 100 KB), valuable, and easy to back
up. Storage is large, less valuable per byte, and impractical to back
up.

## 6. Capacity envelope

The capacity envelope captures the realistic shape of earnings and
storage utilisation over a node's life.

### Utilisation curve

| Phase                | Time after authorization | Disk filled         | Earnings shape                  |
|----------------------|--------------------------|---------------------|---------------------------------|
| Vetting              | 0 to ~30 days            | Below 5% of allocation  | Near-zero                        |
| Post-vetting ramp    | 1 to 6 months            | Growing 5 to 60%        | Slow ramp, dominated by ingress  |
| Mature node          | 6 to 24 months           | 60 to 95%           | Stable, dominated by egress      |
| Saturated node       | 24+ months               | 95 to 100% then plateau | Egress-dominated, no new ingress |

The plateau is the operating regime: a saturated node earns from egress
of stored pieces and from held-storage accumulation, while new ingress
goes to nodes that still have headroom.

### Vetting-period yield

| Aspect | Value |
|---|---|
| Satellite traffic share during vetting | Under 5% of vetted-node baseline |
| Expected vetting duration              | Roughly 30 days per satellite under stable uptime |
| Expected earnings during vetting       | Near zero; treat as cost of entry |
| Hardware payback start                 | Month 3 to month 6 from authorization, depending on hardware cost |

### Traffic shape post-vetting

| Traffic type    | Approximate share of earnings | Note                                                  |
|-----------------|-------------------------------|-------------------------------------------------------|
| Egress          | 60 to 80%                     | The dominant revenue lever; customer downloads        |
| Held-storage    | 15 to 30%                     | Per-TB-month accrued; rises with stored piece count   |
| Ingress         | 5 to 15%                      | Per-byte uploaded; matters more during fill phase     |
| Repair          | Variable                      | Spikes when peer nodes fail; not a steady revenue line |

The exact shares depend on Storj's customer mix at the time. The
operator does not control this. Track actuals against this envelope
quarterly; deviation outside the bands signals either a problem or a
network-level shift worth understanding.

### Per-cluster capacity context

For an 8-to-10 node cluster targeting 4 to 8 TB per node:

| Metric                              | Range                          |
|-------------------------------------|--------------------------------|
| Total allocated storage              | 32 to 80 TB                    |
| Expected steady-state utilisation    | 60 to 95% (19 to 76 TB filled) |
| Approximate steady-state egress       | Operator-specific; tracking required |
| Approximate steady-state revenue (post-vetting) | Operator-specific; see DE-OPERATOR.md §4 |

No earnings numbers ship in this document. Revenue depends on Storj's
customer mix, EUR-USD rate, electricity cost in the operator's region,
and hardware cost. Operators forecast against their actuals after the
first vetted quarter.

## 7. Trade-off decisions

Each decision below names the alternative and the reason for the choice.

### Decision 1: Single container per node (not separate updater + node)

**Picked:** One `storjlabs/storagenode` container per node, updater built-in.
**Alternative:** Separate `storjlabs/storagenode-updater` container paired
with each storagenode.
**Reason:** Storj bundled the updater into the storagenode image; the
separate updater is no longer published with a `latest` tag. The
single-container model also removes one source of compose complexity.

### Decision 2: Storj container `:latest` tag (not pinned digest)

**Picked:** `storjlabs/storagenode:latest` with digest pinned at
template-publication time but allowed to drift through the built-in
updater.
**Alternative:** Pin to digest and disable the updater.
**Reason:** Storj's operational model assumes auto-update; disabled
updaters fall behind the protocol version, leading to disqualification.
The digest is recorded in this document for reproducibility (the
template was validated against the digest above) but the running
container is allowed to roll forward.

### Decision 3: Bind mounts (not Docker named volumes) for storage

**Picked:** Bind-mount `/opt/storj/storage-N` into the container.
**Alternative:** Docker named volumes.
**Reason:** Direct visibility into the data directory from the host
matters for monitoring, backup, and filesystem-level checks. Named
volumes obscure the underlying path and complicate cross-host migration.

### Decision 4: Compose (not Kubernetes, not Swarm, not bare systemd)

**Picked:** Docker Compose v2.
**Alternative considered:** Kubernetes (k3s for single-host, full k8s for
multi-host), Docker Swarm, bare systemd unit files per container.
**Reason:** 8-to-10 nodes managed by one operator does not earn the
orchestrator overhead of k8s. Compose covers the lifecycle (start, stop,
restart, log, exec) without requiring cluster operators. Swarm is
abandoned upstream. Bare systemd works but loses compose's parallel
lifecycle commands.

### Decision 5: node-exporter as a container (not host install)

**Picked:** Run `prom/node-exporter` as a container alongside storagenode
containers.
**Alternative:** Install node-exporter as a systemd service on the host.
**Reason:** Container parity simplifies the compose file and the
upgrade path. Host-install also works but couples node-exporter version
management to the host's package manager.

### Decision 6: Storagenode native metrics endpoint (not separate exporter)

**Picked:** Prometheus scrapes the storagenode's built-in debug endpoint
at `127.0.0.1:5999/metrics`.
**Alternative:** Run a separate community `storagenode-exporter`
sidecar that translates storagenode logs into Prometheus metrics.
**Reason:** The native endpoint is documented, stable, and exposes the
operationally important metrics directly. Separate exporters add a
moving part and a translation layer.

### Decision 7: Prometheus pull (not push gateway)

**Picked:** Prometheus pulls from node-exporter and storagenode endpoints.
**Alternative:** Storagenode and node-exporter push to a pushgateway,
Prometheus scrapes the pushgateway.
**Reason:** Pushgateway is for batch jobs and short-lived services.
Storagenode and node-exporter are long-running; pull is the correct
Prometheus pattern for them.

### Decision 8: One Grafana per operator, not per cluster

**Picked:** One Grafana instance on the operator's admin host, all
nodes scraped by one Prometheus.
**Alternative:** Per-cluster or per-location Grafana instances.
**Reason:** A solo operator does not run multiple observability stacks
in parallel. Single-pane-of-glass is the load-bearing operational
property for a one-person operation.

## 8. Cross-references

- Operations rhythm and SLO: [OPERATIONS.md](./OPERATIONS.md)
- Host-side hardening profile: [HARDENING.md](./HARDENING.md)
- DE regulatory perimeter: [DE-OPERATOR.md](./DE-OPERATOR.md)
- Compose template + framing: [compose/README.md](./compose/README.md)
- Monitoring template + framing: [monitoring/README.md](./monitoring/README.md)
- Cross-architecture context (Storj vs Akash vs CSM):
  [operator-references/DOCTRINE.md](https://github.com/mj-deving/operator-references/blob/main/DOCTRINE.md)
- DePIN-operator decision-surface comparison:
  [operator-references/COMPARISON.md](https://github.com/mj-deving/operator-references/blob/main/COMPARISON.md)
  (in flight at v0.1; ships in a later commit)

## Open questions for v0.2

- Whether to ship a hardened systemd-as-pid1 variant of the compose
  (some operators prefer `Restart=always` over `restart: unless-stopped`)
- Whether to add an IPv6-only example for operators on IPv6-native
  fibre providers
- Whether to document the migration path from `storj/storagenode-docker`
  (the older combined image) to `storjlabs/storagenode` for operators
  upgrading from pre-2024 setups
