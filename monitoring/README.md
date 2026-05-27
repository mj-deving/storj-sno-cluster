# monitoring/

> **Status:** Scaffolding only. Placeholders for endpoints, retention,
> alert thresholds, and PromQL metric names. Never validated against
> live metrics. Storj metric names exposed by the storagenode debug
> endpoint change between releases; verify against your running
> version's `/metrics` output before tuning alerts or wiring panels.

This directory contains a Prometheus configuration, alert rule
definitions, and a minimal Grafana dashboard JSON for the Storj SNO
cluster. None of the files are runnable as published; each contains
`<PLACEHOLDER>` markers where operator-specific values belong.

## Files

| File                                | Purpose                                                       |
|-------------------------------------|---------------------------------------------------------------|
| `prometheus.yml`                    | Scrape configs for node-exporter + storagenode debug endpoint |
| `alerts.yml`                        | Three tiers of alert rules (P1 / P2 / P3) per OPERATIONS.md   |
| `grafana/dashboard-storj.json`      | Six-panel reference dashboard (uptime, ingress, egress, audit, suspension, disk) |

Prometheus + Grafana containers themselves are out of scope for this
template. Use the digest-pinned versions documented in `ARCHITECTURE.md`
or the operator's existing observability stack.

## Why template-only

Three reasons:

1. **No live node exists to feed the stack.** Wiring dashboards against
   no data source produces a "looks plausible" artifact that bears no
   relationship to actual operations. Half-runnable monitoring is worse
   than fully-template; the operator does not gain reliable signal from
   a stack that was never observed against live data.
2. **Storj metric names depend on the storagenode version.** The debug
   endpoint exposes different metric names across releases; pinning
   PromQL queries to a specific metric name at template-publication time
   would mislead operators on newer versions.
3. **Per-operator thresholds matter more than templates.** Alert
   thresholds (when does a node count as "down"? what suspension score
   triggers a page?) are operator-specific. The template provides the
   structure (three tiers, named channels) but leaves the numbers to
   the operator.

## Operator-side resolution checklist

Before deploying this stack against live nodes, the operator must
resolve every placeholder. Search each file for `<` (left angle
bracket) and confirm none remain:

```bash
grep -rEn '<[A-Z_]+>' monitoring/ || echo "no placeholders remain"
```

### Placeholders organised by category

| Category               | Examples                                                  | Where to resolve                              |
|------------------------|-----------------------------------------------------------|-----------------------------------------------|
| Network endpoints      | `<NODE_HOST_1_MESH_IP>`, `<ALERTMANAGER_HOST>`            | `prometheus.yml`                              |
| Scrape intervals       | `<EXAMPLE_SCRAPE_INTERVAL>`, `<EXAMPLE_EVAL_INTERVAL>`    | `prometheus.yml`                              |
| Cluster labels         | `<CLUSTER_LABEL>`, `<OPERATOR_HANDLE>`                    | `prometheus.yml`                              |
| Storj metric names     | `<STORJ_SUSPENSION_METRIC>`, `<STORJ_AUDIT_SCORE_METRIC>`, etc. | `alerts.yml`, `grafana/dashboard-storj.json` |
| Thresholds             | `<EXAMPLE_SUSPENSION_THRESHOLD>`, `<EXAMPLE_DISK_FULL_THRESHOLD>` | `alerts.yml`                                  |
| Alert routing channels | `<PAGE_CHANNEL>`, `<WARN_CHANNEL>`, `<DIGEST_CHANNEL>`    | `alerts.yml` (then wire into alertmanager)    |
| Grafana datasource     | `<PROMETHEUS_DATASOURCE_UID>`                             | `grafana/dashboard-storj.json`                |
| Node identities        | `<NODE_INSTANCE>`, `<NODE_ID>`, `<SATELLITE_ID>`          | `grafana/dashboard-storj.json`                |
| Dashboard meta         | `<DASHBOARD_UID>`, `<OPERATOR_TIMEZONE>`, `<EXAMPLE_REFRESH_INTERVAL>` | `grafana/dashboard-storj.json`                |

## Recommended deployment shape

For a solo operator running this monitoring stack alongside the storj
nodes:

```
operator admin host
├── prometheus (docker, on operator mesh)
├── grafana (docker, on operator mesh, behind authenticated reverse proxy)
├── alertmanager (docker, on operator mesh, routes to operator's actual
│                 notification surfaces: Telegram / Pushover / email)
└── wireguard or tailscale interface to each storagenode host
```

Storagenode hosts expose `:9100` (node-exporter) and `:5999` (storagenode
debug) on the mesh interface only. The HARDENING.md UFW rules deny
these ports from the public internet.

## Discovery flow for metric names

The metric names referenced by `<STORJ_*_METRIC>` placeholders are
discovered, not specified by this template:

```bash
# On a storagenode host, query the debug endpoint directly
curl -s http://127.0.0.1:5999/metrics | grep -E '^storj_' | awk '{print $1}' | sort -u
```

The list returned by your storagenode version is the authoritative
metric vocabulary. Substitute the actual names into `alerts.yml` and
`grafana/dashboard-storj.json` before deployment.

## What this stack does not include

- `alertmanager.yml` for routing (operator's actual channels vary too
  widely to template usefully)
- Long-retention TSDB configuration (operator-side TSDB sizing decision)
- Multi-cluster federation (single operator, single cluster scope)
- SLO burn-rate alerts (the storj SNO model does not surface a clean SLO
  surface for hand-rolled burn-rate computation; satellite scoring is
  the closest equivalent)
- Synthetic external probing (out of scope; storj satellites perform
  this role at the protocol level)
