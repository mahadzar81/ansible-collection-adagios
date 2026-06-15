# mahadzar81.adagios — Ansible Collection

[![CI](https://github.com/mahadzar81/ansible-collection-adagios/actions/workflows/ci.yml/badge.svg)](https://github.com/mahadzar81/ansible-collection-adagios/actions/workflows/ci.yml)
[![Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-mahadzar81.adagios-blue)](https://galaxy.ansible.com/mahadzar81/adagios)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Ansible collection to deploy **Nagios Core 4.x** and **Adagios** web UI with:

- Full **SELinux support** for RHEL 8/9 and Rocky Linux (file contexts, booleans, custom TE policy)
- **Idempotent** task design — safe to re-run without side effects
- **MK Livestatus** broker module for live status queries
- **PNP4Nagios** for performance graphing
- GitHub Actions for CI (lint + Molecule) and automated release to Galaxy

---

## Requirements

| Component        | Version   |
|-----------------|-----------|
| Ansible          | >= 2.9    |
| ansible.posix    | >= 1.4.0  |
| community.general| >= 6.0.0  |

Install collection dependencies:
```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Supported Platforms

| OS                  | SELinux Support |
|--------------------|----------------|
| RHEL 8 / 9          | ✅ Full         |
| Rocky Linux 8 / 9   | ✅ Full         |
| AlmaLinux 8 / 9     | ✅ Full         |
| Ubuntu 20.04 / 22.04| N/A (no SELinux)|
| Debian 10 / 11      | N/A (no SELinux)|

---

## Roles

### `nagios_core`

Compiles and installs **Nagios Core** from source.

| Variable                  | Default                        | Description                              |
|--------------------------|-------------------------------|------------------------------------------|
| `nagios_version`          | `4.5.0`                       | Nagios Core version to compile           |
| `nagios_plugins_version`  | `2.4.8`                       | Nagios Plugins version                   |
| `nagios_home`             | `/usr/local/nagios`           | Installation prefix                      |
| `nagios_user`             | `nagios`                      | System user for Nagios                   |
| `nagios_group`            | `nagios`                      | Primary group                            |
| `nagios_cmdgroup`         | `nagcmd`                      | Command group (Apache must be a member)  |
| `nagiosadmin_pass`        | *(required — no default)*     | htpasswd password for `nagiosadmin`      |
| `nagios_firewall_manage`  | `true`                        | Open HTTP port via firewalld/ufw         |
| `nagios_firewall_zone`    | `public`                      | firewalld zone                           |
| `nagios_selinux_manage`   | `true`                        | Apply SELinux contexts and policy        |
| `nagios_selinux_policy_dir`| `/usr/local/nagios/selinux`  | Where to build the custom .te policy     |

> ⚠️ **Always override `nagiosadmin_pass`** in your vars or Ansible Vault. The role will fail if the default placeholder value is used.

### `adagios`

Installs **Adagios**, **MK Livestatus**, and **PNP4Nagios**.

| Variable                  | Default                        | Description                             |
|--------------------------|-------------------------------|-----------------------------------------|
| `adagios_repo`            | GitHub opinkerfi/adagios      | Git repository URL                      |
| `adagios_version`         | `master`                      | Branch or tag to deploy                 |
| `adagios_install_dir`     | `/opt/adagios`                | Installation directory                  |
| `adagios_venv`            | `/opt/adagios-venv`           | Python virtualenv path                  |
| `livestatus_version`      | `1.2.8p25`                    | MK Livestatus version to compile        |
| `livestatus_broker_dir`   | `/usr/local/lib/nagios/brokers`| Livestatus .o module destination       |
| `pnp4nagios_version`      | `0.6.26`                      | PNP4Nagios version to compile           |
| `adagios_selinux_manage`  | `true`                        | Apply SELinux contexts and policy       |

---

## Quick Start

### 1. Install the collection

```bash
ansible-galaxy collection install mahadzar81.adagios
```

### 2. Create your inventory

```ini
# inventory/hosts.ini
[monitoring]
mon01.example.com
```

### 3. Create a vars file (use Vault for the password)

```yaml
# vars/monitoring.yml
nagiosadmin_pass: "{{ vault_nagiosadmin_pass }}"
nagios_version: "4.5.0"
nagios_selinux_manage: true   # set false on non-SELinux hosts
adagios_selinux_manage: true
```

### 4. Run the playbook

```bash
ansible-playbook playbooks/site.yml \
  -i inventory/hosts.ini \
  -e @vars/monitoring.yml \
  --ask-vault-pass
```

---

## SELinux Details

On RHEL/Rocky Linux with SELinux in **Enforcing** mode, the collection:

1. **Labels** `/usr/local/nagios/**` with appropriate types (`nagios_exec_t`, `nagios_log_t`, `nagios_etc_t`, `nagios_rw_t`, `nagios_var_run_t`, `httpd_sys_content_t` for the web share).
2. **Sets booleans**: `httpd_run_nagios`, `httpd_can_sendmail`, `httpd_can_network_connect`, etc.
3. **Compiles and loads** a custom Type Enforcement (`.te`) policy module to allow:
   - Apache (`httpd_t`) to write to the Nagios external command pipe.
   - Apache to connect to the MK Livestatus Unix socket.
   - Apache to execute CGI scripts from `/usr/local/nagios/sbin`.
   - Nagios to execute plugins and write logs.

All SELinux tasks are **idempotent**: file contexts use `sefcontext` (which tracks state), `restorecon` only runs when contexts change, booleans are applied with `persistent: true`, and the custom policy module is only recompiled when the `.te` template changes.

To **disable** SELinux management (e.g., on Permissive hosts or non-SELinux systems):

```yaml
nagios_selinux_manage: false
adagios_selinux_manage: false
```

---

## Idempotency Design

Every potentially non-idempotent operation is guarded:

| Technique                   | Used for                                              |
|----------------------------|-------------------------------------------------------|
| **Version stamp files**     | Nagios Core, Nagios Plugins, Livestatus, PNP4Nagios  |
| `creates:` on `command`     | configure, make, make install steps                   |
| `ansible.builtin.git`       | Adagios — tracks exact commit/version                 |
| `ansible.builtin.pip`       | Virtualenv packages — pip handles state               |
| `community.general.htpasswd`| nagiosadmin password — only updates if changed        |
| `community.general.sefcontext`| SELinux file contexts — tracks present/absent       |
| `ansible.posix.seboolean`   | SELinux booleans — no-op if already set               |
| `.te` template change check | Custom policy only recompiles when template changes   |
| `lineinfile`                | Nagios broker_module config                           |

---

## Releasing

Releases are fully automated via GitHub Actions:

1. Bump `version:` in `galaxy.yml`.
2. Commit and push.
3. Push a matching tag:
   ```bash
   git tag v1.2.0
   git push origin v1.2.0
   ```
4. The `release.yml` workflow automatically:
   - Validates the tag matches `galaxy.yml`.
   - Builds the `.tar.gz` collection artifact.
   - Creates a **GitHub Release** with auto-generated changelog.
   - Publishes to **Ansible Galaxy**.

> **Required secret**: Add `GALAXY_API_KEY` to your repository secrets (Settings → Secrets → Actions).

---

## Development

### Running Molecule tests locally

```bash
pip install molecule molecule-docker ansible ansible-lint docker
ansible-galaxy collection install -r requirements.yml
molecule test
```

### Linting

```bash
yamllint .
ansible-lint
```

---

## License

MIT
