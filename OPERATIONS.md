# OPERATIONS

Reference operations profile for a Storj SNO multi-node setup running
the compose stack in this repository. Targets the eight-to-ten-node
cluster shape on a single host or a small cluster of hosts owned by
one operator.

> **Status:** Reference profile. Each section names what to watch,
> when to act, and how to verify. The profile is written for a solo
> operator without a 24/7 oncall rotation, alerting and lifecycle
> choices reflect that.

## What this profile is

- A documented operational rhythm for a solo SNO
- Named checks at daily, weekly, monthly cadence
- A pointer to the metrics that matter for the SNO's per-node SLO
  (uptime, audit score, suspension score, egress)
- A backup and recovery posture for the irreplaceable per-node state

## What this profile is not

- A fleet-operations runbook (no shift handoffs, no incident-commander
  protocol; a solo operator does not run a NOC)
- A guarantee of any particular earnings level
- A substitute for the Storj forum and docs when a node misbehaves;
  the community there is the operational ground truth

---

## 1. Monitoring philosophy

Three categories of signal, in priority order.

### Category A: node health (must-watch)

- **Online status.** Is the storagenode container running? Is it
  reachable on TCP and UDP port 28967 from outside?
- **Audit score.** Per-satellite audit score; declining audits drift
  a node toward disqualification.
- **Suspension score.** Per-satellite suspension score; below 60% on
  any satellite means the node is suspended.
- **Disk usage.** Per-node disk used vs allocated. Filling to the
  ceiling stops ingress but does not by itself disqualify.

### Category B: workload signal (watch trend, not single values)

- **Egress traffic.** Customer downloads, the dominant earnings
  driver after vetting.
- **Ingress traffic.** Customer uploads plus repair traffic.
- **Pieces stored.** Total piece count per node and per satellite.
- **GC (garbage collection) progress.** Garbage collection cycles
  completing on schedule.

### Category C: host signal (cross-watch with the host's own monitoring)

- CPU, memory, disk I/O
- Network throughput at the host NIC
- System log volume (a spiking syslog is a sign of trouble)
- Container restart count

## 2. What wakes you up at 3 AM (alerting policy)

Three alert tiers. The thresholds are advisory; tune to taste.

### P1: page immediately

- **Node down.** Any storagenode container exited or unreachable for
  more than 10 minutes.
- **Suspension score below 70% on any satellite.** Recoverable but
  time-sensitive.
- **Disk full.** Any node's data volume above 95% utilisation;
  ingress stops above 100%.

### P2: alert next morning

- **Audit score declining for three consecutive scrape windows on any
  satellite.** Not yet failing but trending wrong.
- **Container restart loop.** Container restarting more than three
  times in 30 minutes.
- **Host disk pressure.** Root filesystem above 85%.

### P3: log and review weekly

- Egress steady-state outside the per-node baseline by more than 50%
- Pieces stored counter not advancing across two scrape windows
- Memory pressure on the host (swap above 10% of physical RAM)

### How alerts reach the operator

For a solo operator without an oncall service:

- **P1** routes to Telegram, Signal, or Pushover. Wake-the-operator
  channel.
- **P2** routes to email. Read in the morning.
- **P3** routes to a weekly digest, generated from the monitoring
  stack on Sunday evening.

The alerting stack itself is template-only in this repository (see
`monitoring/`). Wire it to the operator's actual notification surface
during deployment.

## 3. Backup posture

Two surfaces matter. Everything else is replaceable.

### Identity files (irreplaceable)

Each per-node identity directory contains:

- `identity.cert`, `identity.key`
- `ca.cert`, `ca.key`

Lose these and the node's history is gone. The node cannot be
re-created from the same identity.

Backup posture:

1. **Encrypted offline copy** of every node's identity directory.
   Recommended encryption: `age` or `gpg`.
2. **Two physical locations**, at least one not in the same building
   as the host.
3. **Restore drill quarterly**, restore one node's identity to a
   throwaway host and confirm the directory is readable. Without a
   verified restore the backup is theoretical.

### Storagenode database (recoverable but expensive to lose)

The storagenode service stores its piece metadata in a local SQLite
database under `storagenode/storage/`. Loss of this database does
not delete pieces but forces a re-walk from disk, which can take
many hours on multi-terabyte nodes. During the re-walk the node is
effectively offline.

Backup posture: snapshot the storage directory while the container
is stopped, weekly. Daily is overkill; never is too rare.

### Configuration (replaceable: version-control it)

`compose/docker-compose.yml`, `.env` (without secrets), and
`monitoring/` config files live in this repository. Per-host
overrides land in `docker-compose.override.yml`, which is
git-ignored. Keep a separate encrypted copy of each host's override.

## 4. Lifecycle: from staging to decommission

### Stage 1: staging

A node is staged but not yet visible to satellites.

- Hardware provisioned and burn-tested for 24 hours under load
- Operating system installed, patched, hardened per `HARDENING.md`
- Docker installed (rootless preferred)
- Identity generated and authorized via Storj's auth process
- Storagenode container deployed but not yet allocated significant
  storage

Exit criterion: storagenode is `Status: ok` in the dashboard and
identity is authorized on at least one satellite.

### Stage 2: vetting

The node is online and receiving the reduced traffic that satellites
send to unvetted nodes (under 5% of vetted-node traffic per
satellite).

- Expected duration: roughly 30 days per satellite under stable
  conditions
- Earnings near zero; this is the cost of entry
- Operator's job: keep the node up. Single outages over a few hours
  extend the vetting period

Exit criterion: vetted status across at least one satellite (visible
on the dashboard).

### Stage 3: full operation

The node receives full traffic from vetted satellites.

- Earnings ramp over the next several months as storage fills
- Operator's job shifts to maintenance: patches, log review,
  monitoring trend analysis, occasional database compactions
- The node compounds: existing pieces accrue held storage rewards;
  new ingress adds to the held-storage base

### Stage 4: suspension recovery (only if triggered)

If a node is suspended on any satellite, the operator has limited
time to fix the cause before disqualification.

- Common causes: persistent uptime below SLA, audit failures from
  disk corruption, missed deletions
- Resolution path: identify the cause from logs, fix the underlying
  issue, wait for the satellite to re-vet
- Recovery window: typically days, occasionally weeks

### Stage 5: graceful exit (decommission)

When the operator decides to decommission a node, the **graceful
exit** flow returns held pieces to the network in exchange for
release of the held escrow.

- Initiated via the storagenode CLI
- Duration: weeks to months depending on stored data volume
- During graceful exit the node continues to serve egress for
  existing pieces; ingress stops
- On completion the held escrow is released and the node deactivates

Exit criterion: graceful exit complete across all satellites; node
container can be removed.

## 5. Operational rhythm

### Daily (5 minutes)

- Check the monitoring dashboard for P1 alerts overnight
- Glance at per-node online status
- Glance at the worst per-satellite audit score across all nodes

### Weekly (30 minutes)

- Review per-node disk usage trend; reallocate storage if any node
  approached 90%
- Skim the storagenode logs for new error patterns
- Confirm the backup of identity directories ran and produced
  the expected file count
- Review the P3 digest

### Monthly (2 hours)

- Reconcile earnings vs estimate; track per-node EUR receipts
- Patch hosts (unattended-upgrades handles security; the operator
  handles minor-version OS upgrades manually)
- Review the suspension and audit score trends month-over-month
- Restore-drill one node's identity to a throwaway host (rotate
  which node each month)
- Settle any STORJ-to-EUR conversions and record in the bookkeeping
  set (see `DE-OPERATOR.md` section 6)

### Quarterly (half a day)

- Compact storagenode databases offline (container stopped, sqlite
  `VACUUM`)
- Re-evaluate hardware: aging disks above 50,000 power-on hours,
  warranty status, replacement budget
- Review the per-node profitability; decommission unprofitable nodes
  via graceful exit

## 6. What this profile deliberately omits

- Multi-host orchestration (Swarm, Kubernetes). The eight-to-ten-node
  target fits on one to two hosts. Orchestration overhead is not
  earned at this scale.
- 24/7 oncall scheduling. A solo operator is not a NOC; alerting
  thresholds reflect this.
- Cross-operator coordination. Each operator runs their own
  infrastructure; this is by Storj's design.
- Earnings forecasting and rate analysis. Earnings are a function of
  Storj's customer base and network competition, neither of which
  the operator controls. Track actuals, not forecasts.

## 7. References

- Storj Node Operator documentation: https://docs.storj.io/node
- Storagenode dashboard and metrics: https://docs.storj.io/node/dashboard
- Vetting period documentation: https://storj.dev/dcs/concepts/satellites/vetting
- Graceful exit documentation: https://docs.storj.io/node/graceful-exit
- Storj community forum: https://forum.storj.io/
