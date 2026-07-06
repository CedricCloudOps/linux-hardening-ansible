# linux-hardening-ansible

An Ansible role that applies a baseline security hardening to Debian/Ubuntu
Linux servers. Idempotent, variable-driven, and safe to re-run.

Author: **Cedric Severin DJIGUIMDE**

## What it does

| # | Control | Details |
|---|---------|---------|
| 1 | System updates | Updates the package cache and applies upgrades |
| 2 | Security tooling | Installs `ufw`, `fail2ban`, `unattended-upgrades` |
| 3 | Least privilege | Creates a non-root **admin user** with sudo + SSH key |
| 4 | SSH hardening | Disables root login & password auth, enforces key-based auth, limits `MaxAuthTries`, disables X11 forwarding (config is validated before reload) |
| 5 | Host firewall | UFW: default **deny inbound** / allow outbound, opens only the required ports |
| 6 | Brute-force protection | `fail2ban` jail for SSH (ban time + max retries) |
| 7 | Auto security updates | Enables `unattended-upgrades` |
| 8 | Kernel/network sysctl | rp_filter, syncookies, ignore ICMP broadcasts, no source routing / redirects |

## Requirements

```bash
ansible-galaxy collection install ansible.posix community.general
```

## Usage

1. Edit `inventory.ini` and add your servers.
2. Set your variables (at least your **SSH public key**) in
   `roles/hardening/defaults/main.yml` or in `group_vars/`.
3. Dry-run first, then apply:

```bash
# Preview the changes without applying them
ansible-playbook hardening.yml --check --diff

# Apply
ansible-playbook hardening.yml
```

## ⚠️ Safety first

- **Set `hardening_admin_ssh_key` correctly before running.** The role
  disables SSH password authentication and root login — without a working
  key you can lock yourself out. Test in a lab/VM (or keep console access)
  first.
- Always run `--check --diff` before applying to production.

## Adapting to RHEL / CentOS

The role targets Debian/Ubuntu. For RHEL/CentOS, replace:
- `apt` / `unattended-upgrades` → `dnf` + `dnf-automatic`
- `ufw` (`community.general.ufw`) → `firewalld` (`ansible.posix.firewalld`)

## Variables (defaults)

See `roles/hardening/defaults/main.yml`. Key ones:

| Variable | Default | Description |
|----------|---------|-------------|
| `hardening_admin_user` | `cedric` | Non-root admin account to create |
| `hardening_admin_ssh_key` | *(placeholder)* | Your SSH **public** key |
| `hardening_ssh_port` | `22` | SSH port |
| `hardening_allowed_tcp_ports` | `[22, 80, 443]` | Inbound ports to open |
| `hardening_fail2ban_bantime` | `3600` | Ban duration (seconds) |
