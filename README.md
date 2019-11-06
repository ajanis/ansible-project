# Ansible-Project

**This role is part of the [ITC Automated Build Pods Project][]**
  - Sets up a base ansible environment for Build-Pods
  - Installs role dependencies
  - Installs ansible-bender

[TOC]

## Dependencies

The role itself has no dependencies. However it is intended to be used as part of the [ITC Automated Build Pods Project][] which requires additional Ansible roles and configurations.

## Requirements

 * The [ITC Automated Build Pods Project][] git repository is included in the role's defaults/main.yml file.

## Role Variables

### defaults/main.yml
```yaml
ansible_user: root
ansible_default_project: https://github.com/ajanis/wwt-build-pods.git
github_deploy_key: "{{ vault_github_deploy_key }}"

build_pkgs:
  - git
  - gcc
  - make
  - g++

```

### vars/debian.yml
```yaml
ansible_python_pkgs:
  - python3
  - python3-pip

ansible_pip_pkgs:
  - ansible
  - ansible-bender

ansible_bender_pkgs:
  - python3-pip
  - software-properties-common
  - uidmap
  - buildah
  - podman

ansible_bender_pkgs_ppa: "ppa:projectatomic/ppa"
```

## Example Playbook
```yaml
- name: "Deploy Build-Pod Environment"
  hosts: all
  remote_user: root
  tasks:

    - include_role:
        name: common
      tags:
        - common
        - packages
        - cloudinit
        - ssh
        - sol
        - sudo
        - autoupdate
        - hostfile
        - timezone

    - include_role:
        name: isc_dhcp_server
        public: yes
      when: foreman_proxy_dhcp

    - include_role:
        name: tftp
        public: yes
      when: foreman_proxy_tftp

    - include_role:
        name: nginx
        public: yes

    - include_role:
        name: awx
        public: yes

    - include_role:
        name: docker
        public: yes

    - include_role:
        name: awx
        tasks_from: update_ca.yml
        public: yes

    - include_role:
        name: foreman
        public: yes
      tags:
        - install
        - configure
        - foreman
        - smartproxy
```

## License

MIT

## Author Information

Created by Alan Janis

[itc automated build pods project]:  https://github.com/ajanis/itc-build-pods.git
