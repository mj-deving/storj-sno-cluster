# HARDENING

Reference hardening profile for a Storj SNO host running the compose stack
in this repository. Targets a single-author Debian or Ubuntu LTS host
running rootless Docker. The profile is opinionated and conservative;
fork and adjust to the host distribution and threat model that actually
apply.

> **Status:** Reference profile. Each prescription below is testable on
> a clean host with the named command. No prescription is generic
> security theatre, every rule maps to a Storj-specific or
> Docker-host-specific attack surface.

## What this profile is

- A minimum baseline for a host that runs the storagenode container
- Documented per-prescription with the command that verifies the state
- Scoped to host-side hardening; the storagenode container itself is
  hardened upstream by Storj Labs

## What this profile is not

- A pen-test report
- A full CIS Benchmark walkthrough (use the upstream CIS Debian or
  Ubuntu profile for that surface)
- A substitute for OS-level patching discipline
- A guarantee against compromise; layered defence is the goal, not
  invulnerability

## Threat surface: named explicitly

| Vector                          | Mitigation in this profile                       |
|---------------------------------|--------------------------------------------------|
| SSH brute force                 | key-only auth, fail2ban, non-default port allowed |
| Unpatched kernel CVE            | unattended-upgrades with reboot policy           |
| Storagenode log credential leak | log rotation, restricted log file mode           |
| Container escape                | rootless Docker, AppArmor, no `--privileged`     |
| Open port enumeration           | UFW default-deny, allow list with port 28967 only|
| Stolen identity files           | identity dir mode 0700, host backup encrypted    |

---

## 1. UFW firewall: default-deny with explicit allow list

The host firewall denies everything by default and allows only the
ports the Storj stack actually needs.

```bash
# Reset to a known state
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH: allow from a small set of trusted source IPs if you have them
sudo ufw allow from <YOUR_ADMIN_IP>/32 to any port 22 proto tcp
# Otherwise: allow SSH from anywhere and lean harder on fail2ban + keys
# sudo ufw allow 22/tcp

# Storj storagenode: inbound TCP and UDP on 28967
sudo ufw allow 28967/tcp
sudo ufw allow 28967/udp

# Monitoring (optional: only if you expose Prometheus or Grafana publicly,
# which you should not without an authenticated reverse proxy)
# sudo ufw allow from <YOUR_MONITORING_NETWORK> to any port 9090 proto tcp

sudo ufw enable
sudo ufw status numbered
```

**Verification:**

```bash
sudo ufw status verbose | grep -E '(Status|Default|28967)'
```

Expected output names `Status: active`, both `Default: deny (incoming)`
and `allow (outgoing)`, and the explicit 28967 allow rules.

**What is not on this list:** every other port. Including 9090
(Prometheus), 3000 (Grafana), 9100 (node-exporter). Monitoring
endpoints belong on a VPN, a Wireguard mesh, or behind an
authenticated reverse proxy. Never exposed raw.

---

## 2. SSH: key-only authentication, no password fallback

`/etc/ssh/sshd_config.d/99-hardening.conf` (preferred over editing the
main file):

```
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
PubkeyAuthentication yes
AuthenticationMethods publickey
PermitEmptyPasswords no
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30
MaxAuthTries 3
AllowUsers <YOUR_ADMIN_USER>
```

Reload:

```bash
sudo sshd -t && sudo systemctl reload ssh
```

**Verification:**

```bash
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|pubkeyauthentication|authenticationmethods)'
```

Every line must match the config above. If `PasswordAuthentication yes`
appears anywhere in the output, a drop-in is being shadowed.

---

## 3. Unattended-upgrades with documented reboot policy

```bash
sudo apt install -y unattended-upgrades apt-listchanges
sudo dpkg-reconfigure -plow unattended-upgrades
```

`/etc/apt/apt.conf.d/50unattended-upgrades`:

```
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::Package-Blacklist {
};

Unattended-Upgrade::Mail "root";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
```

**Reboot policy choice:** automatic reboot at 04:00 local. A Storj
node missing a reboot for a kernel CVE is worse than a node offline
for 90 seconds. The vetting period is forgiving of short outages;
unpatched kernels are not.

**Verification:**

```bash
sudo unattended-upgrade --dry-run --debug | tail -20
```

The dry-run must print the configured allowed origins and the chosen
reboot time.

---

## 4. fail2ban: SSH and storagenode log monitoring

```bash
sudo apt install -y fail2ban
```

`/etc/fail2ban/jail.d/sshd.local`:

```
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = %(sshd_log)s
backend  = systemd
maxretry = 3
findtime = 10m
bantime  = 1h
```

`/etc/fail2ban/jail.d/storagenode.local` (optional, if log volume
warrants, high false-positive rate, treat as advisory):

```
[storagenode-error]
enabled  = false
port     = 28967
filter   = storagenode-error
logpath  = /var/log/storj/storagenode-*.log
maxretry = 50
findtime = 5m
bantime  = 30m
```

The storagenode jail is disabled by default in this profile. It is
listed for completeness; enabling it requires authoring a matching
filter in `/etc/fail2ban/filter.d/storagenode-error.conf`. Most
operators find SSH-only fail2ban sufficient.

**Verification:**

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

`status sshd` must show `Enabled: True` and a nonzero `Currently
banned` after the first day if the host has any public exposure.

---

## 5. AppArmor: docker-default profile

The `docker-default` AppArmor profile is applied automatically by
the Docker daemon. The host-side requirement is that AppArmor itself
is installed, enabled, and not in complain mode globally.

```bash
sudo apt install -y apparmor apparmor-utils
sudo systemctl enable --now apparmor
```

**Verification:**

```bash
sudo aa-status | head -5
docker info 2>/dev/null | grep -E 'Security Options'
```

`aa-status` must print `apparmor module is loaded` and a profile count
greater than zero. `docker info` must list `apparmor` under Security
Options.

Custom per-container profiles are out of scope for this reference;
the upstream Storj storagenode runs cleanly under `docker-default`.

---

## 6. Identity files and host backup posture

Storj identity files (`identity.cert`, `identity.key`,
`ca.cert`, `ca.key`) are the irreplaceable per-node secret. Lose them
and the node's history is gone with them.

Per-node identity directory mode and ownership:

```bash
sudo chmod 0700 /opt/storj/identity
sudo find /opt/storj/identity -type f -exec chmod 0600 {} \;
sudo chown -R <YOUR_OPERATOR_USER>:<YOUR_OPERATOR_USER> /opt/storj/identity
```

Backup posture (advisory, not prescribed):

- Encrypted offline copy of each node's identity directory
- Storage location separated from the host running the node
- Quarterly restore drill on a throwaway host to confirm the backup is
  readable

**Verification:**

```bash
stat -c '%a %n' /opt/storj/identity
```

Must return `700 /opt/storj/identity`.

---

## What this profile deliberately omits

- Kernel hardening beyond stock Debian or Ubuntu defaults
  (sysctl `net.ipv4.tcp_syncookies` and similar are already on by default)
- SELinux configuration (the profile targets AppArmor-based distributions)
- Auditd rule sets (out of scope for a single-host reference; meaningful
  at fleet scale, overkill for one to ten nodes)
- HIDS and FIM tooling (osquery, Wazuh, Tripwire), operationally
  expensive for a solo operator and not load-bearing for the threat
  model named above
- Per-container custom AppArmor profiles
- Hardware root-of-trust (TPM-backed disk encryption); recommend it,
  do not prescribe it because it is hardware-dependent

## References

- Debian Security Manual: https://www.debian.org/doc/manuals/securing-debian-manual/
- Ubuntu Security: https://ubuntu.com/security
- Docker security: https://docs.docker.com/engine/security/
- Docker rootless: https://docs.docker.com/engine/security/rootless/
- AppArmor on Ubuntu: https://ubuntu.com/server/docs/security-apparmor
- fail2ban manual: https://github.com/fail2ban/fail2ban/wiki
