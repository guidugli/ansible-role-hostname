[![CI](https://github.com/guidugli/ansible-role-hostname/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-hostname/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/release/guidugli/ansible-role-hostname)](https://github.com/guidugli/ansible-role-hostname/releases)
[![Galaxy](https://img.shields.io/ansible/role/guidugli/hostname)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/hostname/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-hostname)](https://github.com/guidugli/ansible-role-hostname/blob/main/LICENSE)

# Ansible Role: hostname

Set the system hostname and manage `/etc/hosts` entries on Linux hosts.

## Requirements

- A supported Linux host running on bare metal or a virtual machine.
- Facts must be gathered before the role runs.
- Elevated privileges are required to change the hostname and modify `/etc/hosts`.
- Containerized targets are detected and skipped because hostname changes and `/etc/hosts` management are not reliable in that execution model.

## Variables

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `hostname_name` | `str` | _required_ | Hostname to configure on the target host. |
| `hostname_remove_127_0_1_1` | `bool` | `false` | Remove any `/etc/hosts` entry that starts with `127.0.1.1`. |
| `hostname_hosts` | `list[dict]` | `[]` | Additional `/etc/hosts` entries to enforce. Each item requires `ip` and `alias`. |

Example variable structure:

```yaml
hostname_name: myserver
hostname_remove_127_0_1_1: true
hostname_hosts:
  - ip: 1.2.3.4
    alias: dev1 dev1.dom.local
```

## Example playbook

```yaml
---
- name: Configure host identity
  hosts: servers
  become: true
  vars:
    hostname_name: myserver
    hostname_remove_127_0_1_1: true
    hostname_hosts:
      - ip: 1.2.3.4
        alias: dev1 dev1.dom.local
  roles:
    - role: guidugli.hostname
```

## Molecule testing

The repository uses the shared Molecule converge and verify playbooks referenced by both the default and systemd scenarios.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

## Execution notes

- **Privilege model:** the role never sets `become`; callers must provide privilege externally when running against real hosts.
- **Container behavior:** when `ansible_facts['virtualization_type']` identifies a container runtime, the role skips hostname and `/etc/hosts` changes.
- **Systemd behavior:** the role itself does not manage services, but the repository includes a dedicated systemd Molecule scenario for template-aligned testing.
- **Molecule coverage:** the shipped Molecule scenarios run in Podman containers, so the verify playbook checks the container skip path there; real hostname changes are exercised on non-container targets.

## License

MIT
