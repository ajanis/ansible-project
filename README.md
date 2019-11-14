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
ansible_hostbuild_project: git@github.com:ajanis/wwt-build-pods.git
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
  - ansible-tower-cli

ansible_bender_pkgs:
  - python3-pip
  - software-properties-common
  - uidmap
  - buildah
  - podman
  - ansible

ansible_bender_pkgs_ppa: "ppa:projectatomic/ppa"
```

## Example Playbook
```yaml
- name: "Deploy Foreman Server"
  hosts: buildhost
  remote_user: root
  vars_files:
    - vault.yml
  tasks:

    - name: Wait for server to come online
      wait_for_connection:
        delay: 60
        sleep: 10
        connect_timeout: 5
        timeout: 900

    - include_role:
        name: common
      tags:
        - common

    - include_role:
        name: isc_dhcp_server
        public: yes
        apply:
          tags:
            - dhcp
      when: foreman_proxy_dhcp
      tags:
        - dhcp

    - include_role:
        name: tftp
        public: yes
        apply:
          tags:
            - tftp
      when: foreman_proxy_tftp
      tags:
        - tftp

    - include_role:
        name: nginx
        public: yes
        apply:
          tags:
            - nginx
      tags:
        - nginx

    - include_role:
        name: awx
        public: yes
        apply:
          tags:
            - awx
      tags:
        - awx

    - include_role:
        name: docker
        public: yes
        apply:
          tags:
            - docker
      tags:
        - docker

    - include_role:
        name: awx
        tasks_from: container-tasks.yml
        public: yes
        apply:
          tags:
            - awx
      tags:
        - awx

    - include_role:
        name: foreman
        public: yes
      tags:
        - install
        - configure
        - foreman
        - smartproxy
        - customize

    - include_role:
        name: ansible-project
        public: yes
      tags:
        - never
        - project
        - projectimport
        - projectclone
```

## License

MIT

## Author Information

Created by Alan Janis

[itc automated build pods project]:  https://github.com/ajanis/itc-build-pods.git
