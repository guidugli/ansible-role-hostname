[![CI](https://github.com/guidugli/ansible-role-hostname/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-hostname/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/release/guidugli/ansible-role-hostname)](https://github.com/guidugli/ansible-role-hostname/releases)
[![Galaxy](https://img.shields.io/ansible/role/guidugli/hostname)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/hostname/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-hostname)](https://github.com/guidugli/ansible-role-hostname/blob/main/LICENSE)

# Ansible Role: hostname

Set the system hostname and manage selected `/etc/hosts` entries on Linux hosts. The role is intentionally privilege-neutral, skips hostname and hosts-file mutation in known container runtimes, and keeps all public inputs documented in both defaults and argument specs.

## Requirements

- Ansible Core 2.14 or newer according to role metadata.
- Linux target with gathered facts before the role runs.
- External privilege escalation for real hosts because changing the hostname and editing `/etc/hosts` normally require root-level permissions.
- The `containers.podman` collection, version `>=1.10.0`, for the included Molecule container scenarios.

## Features

- Validates the required hostname and optional hosts-file entries before making changes.
- Sets the runtime and persistent hostname with `ansible.builtin.hostname` on non-container hosts.
- Optionally removes the `127.0.1.1` hosts-file entry when `hostname_remove_127_0_1_1` is `true`.
- Optionally manages deterministic additional `/etc/hosts` entries from `hostname_hosts`.
- Skips hostname and `/etc/hosts` changes in known container runtimes to avoid unreliable container behavior.
- Provides shared Molecule converge and verify playbooks used by both default and systemd scenarios.

## Supported platforms

The role metadata lists Fedora, Ubuntu, and Debian. The bundled shared platform matrix currently includes Ubuntu 26.04 and 24.04, Debian 13 and 12, and Fedora 44 and 43.

## Variables

All public inputs are defined in `defaults/main.yml` and mirrored in `meta/argument_specs.yml`.

| Variable | Type | Default | Required | Description |
| --- | --- | --- | --- | --- |
| `hostname_name` | `str` | No default | Yes | Hostname to configure on the target host. Provide a non-empty string such as `app01` or `app01.example.com`. |
| `hostname_remove_127_0_1_1` | `bool` | `false` | No | When `false`, the role ensures `127.0.1.1 {{ hostname_name }}` exists. When `true`, the role removes entries that begin with `127.0.1.1`. |
| `hostname_hosts` | `list[dict]` | `[]` | No | Additional `/etc/hosts` records to enforce. Each item requires `ip` and `alias`; `alias` may contain one or more host aliases separated by spaces. |

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

## Molecule testing instructions

The repository uses shared Molecule converge and verify playbooks referenced by both the default and systemd scenarios.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

## Execution notes

- **Privilege model:** the role never declares `become`, `become_user`, or `become_method`. Use `become: true` at the play, inventory, or automation-controller level for real hosts where hostname and `/etc/hosts` changes require elevated privileges.
- **Container behavior:** when `ansible_facts['virtualization_type']` is one of the known container values, hostname and `/etc/hosts` mutation tasks are skipped. Molecule containers generally execute as root and do not require role-level privilege escalation.
- **Systemd behavior:** the role does not manage services or unit files. The systemd Molecule scenario is retained from the template so the role continues to exercise the repository-standard scenario layout.
- **Idempotency:** the role uses Ansible modules, deterministic lines, and read-only validation tasks so a second run should report no changes after the desired state is reached.

## Release workflow

Generated metadata and inventories are maintained by the repository generator scripts. Refresh and release with:

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

## License

MIT

## Author

Carlos Guidugli
