# compose/

> **Template.** The YAML structure is valid and the pinned image digests
> resolve on Docker Hub. Requires your own Storj node identity and
> authorization token. The storagenode image digest is pinned at
> template-publication time; the container's built-in auto-updater is
> allowed to roll the binary forward in line with Storj's protocol
> releases (see ARCHITECTURE.md § Decision 2).

This directory contains a Docker Compose template for an 8-to-10-node
Storj SNO cluster on a single host (or scaled across multiple hosts
with one compose stack per host).

## Prerequisites

| Requirement                          | Verify                                    |
|--------------------------------------|-------------------------------------------|
| Docker Engine 24+ (rootless preferred) | `docker --version`                        |
| Docker Compose v2 (`docker compose`) | `docker compose version`                  |
| Wireguard or Tailscale mesh up        | `ip addr show wg0` (or `tailscale status`)|
| Storj signup completed                | Authorization tokens in operator's inbox  |
| Host hardening applied                | See [`../HARDENING.md`](../HARDENING.md)  |

## Step-by-step bring-up

### 1. Generate identities (one per node)

Run on the host where the storj CLI is installed. Repeat for each node:

```bash
# Install the storj CLI; latest release at https://github.com/storj/storj/releases
storj identity create storagenode

# Authorize against a satellite using the auth token from your inbox
storj identity authorize storagenode <auth-token-from-inbox>

# Move the identity directory to the per-node host
sudo mkdir -p /opt/storj/identity-1
sudo cp ~/.local/share/storj/identity/storagenode/* /opt/storj/identity-1/
sudo chmod 0700 /opt/storj/identity-1
sudo chmod 0600 /opt/storj/identity-1/*
sudo chown -R "$USER:$USER" /opt/storj/identity-1
```

### 2. Prepare per-node storage directories

```bash
for i in 1 2; do
  sudo mkdir -p "/opt/storj/storage-$i"
  sudo chown -R "$USER:$USER" "/opt/storj/storage-$i"
done
```

### 3. Populate `.env`

```bash
cp .env.example .env
$EDITOR .env
```

Fill in:

- `WALLET_ADDRESS` (cannot change post-authorization)
- `OPERATOR_EMAIL`
- `UID` / `GID` (from `id -u` / `id -g`)
- `WG_INTERFACE_IP` (the operator-mesh interface address)
- Per-node `NODE_ADDRESS_N`, `NODE_PORT_N`, `STORAGE_ALLOCATION_N`,
  `IDENTITY_DIR_N`, `STORAGE_DIR_N`, `DASHBOARD_PORT_N`, `DEBUG_PORT_N`

### 4. Verify compose syntax

```bash
docker compose -f compose/docker-compose.yml config | head -40
```

Should print the resolved compose structure with substituted env values.

### 5. Bring up

```bash
docker compose -f compose/docker-compose.yml up -d storagenode-1
# Watch the logs until the dashboard reports "Status: ok"
docker compose -f compose/docker-compose.yml logs -f storagenode-1
# Once stable, bring up node 2
docker compose -f compose/docker-compose.yml up -d storagenode-2
# Then node-exporter
docker compose -f compose/docker-compose.yml up -d node-exporter
```

### 6. Confirm dashboard

```bash
curl -s http://127.0.0.1:14002/api/sno/ | jq '.satellites[0].id, .nodeID, .diskSpace'
```

Should return a satellite ID, the node ID, and a disk-space object.

## Scaling beyond 2 nodes

Each additional node requires:

1. A new identity (`storj identity create storagenode` again)
2. A new storage directory on the host
3. A new external port on the NAT (28969, 28970, ...)
4. A new service block in `docker-compose.yml` (copy `storagenode-2`,
   bump the suffix and the env-var indices)
5. New env-var rows in `.env`

The 10-node ceiling is operator-side, not template-side. Beyond ~10
nodes per operator, the operational rhythm (see `../OPERATIONS.md`) and
the IP-diversification target (see `../ARCHITECTURE.md` § 1) both
strain.

## Image digest verification at deploy time

The pinned `sha256:21ca0ac819d1d66240830d28097eeb693923e51383aa3bb07bf823f9fad1f6e9`
was resolved on 2026-05-27 against `storjlabs/storagenode:latest` and
`sha256:4032c6d5bfd752342c3e631c2f1de93ba6b86c41db6b167b9a35372c139e7706`
against `prom/node-exporter:v1.8.2`. To re-resolve at deploy time:

```bash
docker pull storjlabs/storagenode:latest
docker images --digests storjlabs/storagenode | awk '$2=="latest"{print $3}'
```

If the resolved digest differs from the pinned one, the Storj release
moved forward. The storagenode's internal updater handles this in-band;
the digest pin in this template is for reproducibility, not version
locking. The compose `image:` line can be updated to track newer
digests after operator review.

## What this template does not do

- Generate identities (you do that with the storj CLI before bring-up)
- Forward NAT ports (host network or router configuration)
- Wire monitoring (see `../monitoring/`)
- Run on Windows (Docker Desktop on Windows is operationally fragile
  for long-running storage workloads; Linux-only target)

## Troubleshooting basics

| Symptom                                   | First check                                                    |
|-------------------------------------------|----------------------------------------------------------------|
| `Status: setup` and never advances         | Identity directory permissions; must be 0700/0600 owned by UID |
| `QUIC mis-configured` in logs              | UDP port 28967 (or your `NODE_PORT_N`) blocked at NAT          |
| `Failed to start server` immediately       | Storage directory missing or unwritable                        |
| Audit failures within first hour           | Identity-to-storage mismatch (wrong identity mounted)          |
| Dashboard reachable, port not reachable    | Asymmetric NAT; verify both TCP and UDP forwarded               |

For anything else, the Storj community forum at
[forum.storj.io](https://forum.storj.io/) is the ground truth.
