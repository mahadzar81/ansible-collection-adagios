# Changelog

All notable changes to this collection will be documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.2.0] - 2024-07-01

### Added
- Full SELinux support for RHEL 8/9 and Rocky Linux:
  - File context labelling via `community.general.sefcontext` for all Nagios
    and Adagios paths (`nagios_exec_t`, `nagios_log_t`, `nagios_etc_t`,
    `nagios_rw_t`, `nagios_var_run_t`, `httpd_sys_content_t`).
  - SELinux boolean management via `ansible.posix.seboolean`
    (`httpd_run_nagios`, `httpd_can_sendmail`, `httpd_can_network_connect`,
    `httpd_can_network_connect_db`).
  - Custom Type Enforcement (`.te`) policy module for Apache → Nagios command
    pipe writes and Livestatus socket access (compiled and loaded via
    `checkmodule` / `semodule_package` / `semodule`).
  - `nagios_selinux_manage` and `adagios_selinux_manage` booleans to skip
    SELinux tasks on non-SELinux hosts.
- GitHub Actions `release.yml` workflow:
  - Triggers on `v*.*.*` tag pushes.
  - Validates `galaxy.yml` version matches the pushed tag.
  - Builds collection tarball with `ansible-galaxy collection build`.
  - Creates a GitHub Release with auto-generated changelog.
  - Publishes to Ansible Galaxy using `GALAXY_API_KEY` repository secret.
- GitHub Actions `ci.yml` workflow:
  - `yamllint` and `ansible-lint` on every push/PR.
  - Molecule tests on Rocky Linux 8, Rocky Linux 9, and Ubuntu 22.04.
  - Automatic second-run idempotency check (fails if any task is `changed`).
- Molecule test scenario `default` with verify playbook covering Nagios binary,
  config validation, service state, htpasswd, Apache config, Livestatus broker,
  and Adagios virtualenv.
- `requirements.yml` declaring `ansible.posix >= 1.4.0` and
  `community.general >= 6.0.0`.
- Example inventory and example vars file under `examples/`.

### Changed
- All `command` tasks now use `creates:` or version stamp files for idempotency.
- Nagios Core, Nagios Plugins, MK Livestatus, and PNP4Nagios each write a
  `.ansible_*_version` stamp file; compile steps are skipped if the installed
  version matches the requested version.
- `nagiosadmin_pass` now has no default and the `nagios_core` role asserts it
  is set to a non-placeholder value before proceeding.
- Apache config tasks use `notify` handlers so restarts only happen on change.
- Adagios git clone uses `ansible.builtin.git` (tracks exact commit; pip
  install and migrations only run when the repo changes).

### Fixed
- Apache user correctly set to `apache` on RedHat family and `www-data` on
  Debian family.
- Debian Apache config uses `conf-available` + symlink to `conf-enabled`.
- `make install-webconf` guarded by `creates:` to prevent re-running on every
  play.

---

## [1.0.0] - 2023-01-15

### Added
- Initial release: `nagios_core` and `adagios` roles for RedHat and Debian
  family systems.
- MK Livestatus and PNP4Nagios installation support.
