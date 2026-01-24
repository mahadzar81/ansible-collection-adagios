### `ansible-collection-adagios/README.md`
# Adagios Ansible Collection

![Ansible](https://img.shields.io/badge/ansible-2.9%2B-red) ![Nagios](https://img.shields.io/badge/nagios-4.x-blue) ![Adagios](https://img.shields.io/badge/adagios-latest-green) ![Platform](https://img.shields.io/badge/platform-RHEL%20%7C%20Debian-lightgrey)

This collection provides a comprehensive, modular stack for deploying **Nagios Core 4.x** and **Adagios** from source. 

Unlike standard package installations, this collection compiles Nagios Core, Nagios Plugins, and MK Livestatus from source to ensure the latest versions and reliable behavior across different Linux distributions.



## 📦 Collection Contents

This collection includes two primary roles:

| Role | Description |
| :--- | :--- |
| **`nagios_core`** | Installs dependencies, compiles Nagios Core 4.x & Plugins, configures Apache, and sets up the `nagiosadmin` user. |
| **`adagios`** | Installs Adagios (Web UI), compiles MK Livestatus (Broker), and sets up the Python/WSGI environment. |

## ⚙️ Compatibility

This collection is tested and designed for:
* **RedHat Family:** RHEL 7/8/9, CentOS, Rocky Linux, AlmaLinux.
* **Debian Family:** Debian 10+, Ubuntu 20.04+.

## 🚀 Quick Start

### 1. Installation
Clone this repository or install the collection locally:

# From source
```
git https://github.com/mahadzar81/ansible-collection-adagios.git
cd ansible-collection-adagios
ansible-galaxy collection build
ansible-galaxy collection install 
cd ansible-collection-adagios
```
# Using ansible galaxy
```
ansible-galaxy collection install mahadzar81.adagios
```

### 2. Inventory Setup

Create an `inventory` file:
```
[monitoring_servers]
192.168.1.50 ansible_user=root
```

### 3. Playbook Usage

Create a playbook named `monitor_stack.yml`:

```yaml
---
- name: Deploy Monitoring Stack
  hosts: monitoring_servers
  become: yes
  collections:
    - mahadzar81.adagios
  
  vars:
    nagiosadmin_pass: "SuperSecurePassword!"
    nagios_version: "4.5.0"

  roles:
    - nagios_core
    - adagios

```

### 4. Run Deployment

```
ansible-playbook -i inventory monitor_stack.yml

```
---

## 🔧 Role Variables

### Role: `nagios_core`

| Variable | Default | Description |
| --- | --- | --- |
| `nagios_version` | `"4.5.0"` | Version of Nagios Core to compile. |
| `nagios_plugins_version` | `"2.4.8"` | Version of Nagios Plugins to compile. |
| `nagios_user` | `"nagios"` | System user for the Nagios daemon. |
| `nagios_home` | `"/usr/local/nagios"` | Installation prefix. |
| `nagiosadmin_pass` | `"Welcome123"` | **Required.** Password for the Web UI. |

### Role: `adagios`

| Variable | Default | Description |
| --- | --- | --- |
| `adagios_repo` | `github.com/opinkerfi/adagios` | Git URL for Adagios source. |
| `livestatus_version` | `"1.2.8p25"` | Version of MK Livestatus to compile. |
| `pnp4nagios_version` | `"0.6.26"` | Version of PNP4Nagios (Graphing). |

---

## ⚠️ Critical Considerations

### SELinux (RHEL/CentOS/Rocky)

Because this collection installs into `/usr/local/nagios` and `/opt/adagios`, strict SELinux policies may block the web server.

* **Recommendation:** For initial testing, set SELinux to Permissive.
* **Production:** You must generate policies allowing Apache to read/write to the Nagios pipes and configuration directories.

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config

```

### Firewall

Ensure your firewall allows HTTP traffic:

**Firewalld (RHEL):**

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

```

**UFW (Ubuntu):**

```bash
ufw allow 80/tcp

```

### MK Livestatus

Adagios communicates with Nagios via the **MK Livestatus broker**. This role compiles the broker object file (`livestatus.o`) and injects it into `nagios.cfg`. If Adagios cannot see hosts, check that Nagios started successfully and loaded the broker module.

## 📝 License

MIT / BSD

## 👤 Author

Maintained by Mahadzar
