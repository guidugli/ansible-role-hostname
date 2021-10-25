Ansible Role: hostname
=========

An Ansible Role that sets hostname and configure `/etc/hosts` on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

Operating system running on bare metal or on a hypervisor virtualization. This role does not work properly on conteinerized systems.

Role Variables
--------------

**Available variables are listed below, along with default values (see `defaults/main.yml`):**

    #hostname_name: myserver

Hostname to be set on target system.

    #hostname_remove_127_0_1_1: yes

If set to yes/true, removes entries pointing to 127.0.1.1 on `/etc/hosts`.

    #hostname_hosts:
    #  - { ip: 1.2.3.4, alias: 'dev1 dev1.dom.local' }

Add specified entries to `/etc/hosts`.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      vars:
        hostname_name: myserver
        hostname_remove_127_0_1_1: yes
        hostname_hosts:
          - { ip: 1.2.3.4, alias: 'dev1 dev1.dom.local' }
      roles:
         - { role: guidugli.hostname }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
