---
name: Bug Report
about: Report a bug in the collection
title: "[BUG] "
labels: bug
assignees: mahadzar81

---

**Describe the bug**
<!-- A clear description of what the bug is. -->

**To Reproduce**
Steps to reproduce the behaviour:
1. OS / version: (e.g. Rocky Linux 9.3)
2. SELinux mode: (`getenforce` output — Enforcing / Permissive / Disabled)
3. Collection version: (`ansible-galaxy collection list mahadzar81.adagios`)
4. Playbook / role invocation used
5. Error output / AVC denial (`ausearch -m avc -ts recent` if SELinux related)

**Expected behaviour**
<!-- What you expected to happen. -->

**Ansible versions**
```
ansible --version
ansible-galaxy collection list
```

**Additional context**
<!-- Any other context, logs, or screenshots. -->
