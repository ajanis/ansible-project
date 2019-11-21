# wwt-build-pods
**[ITC Automated Build Pods Project][] :: Quickly create a build server custom-tailored to your lab environment**

## Server components:

* Ubuntu 18.04
* Ansible will be installed on this host with a static inventory (NUC hosts) and the “build-server-provisioning” roles and playbooks.
* Most of the other build software is not required on this host since it will only deploy a limited set of hosts and features.
* If YUM or Apt repository mirrors are to be used, this host can act as a mirror to allow for faster local copies to the build-servers mirrors.
* Wide selection of OS distributions, versions and releases can be housed here for faster local copy when pre-seeding Foreman with OS images.
* Ansible Tower could be used if the GUI or other features (such as API, access roles, playbook chaining, logging) could be helpful.

#### Foreman Overview

#### AWX Overview



## Deploying Build Master

#### Ansible Configuration

#### Server Build

#### Web Service Logins



## Deploying Intel NUC Build-Server for a project

#### AWX Configuration
Ansible group_vars will need to be defined for lab/itc build environment:
- Build network VLAN/Subnet
- Build Domain info for local DNS
- DHCP range

Ansible will be used to install and configure various build-server components:
- Foreman (will provide DHCP and DNS)
- Common Foreman build templates
- Common OS Images
- Nginx
- PureFTPD
- iLO/iDRAC/IPMI Access tools
- SSHD
- Ansible
- Library of ansible roles
- Build User shell account
- Build home-directory (Templated home directory for running ansible playbooks or other scripts)
- Python libraries required by ansible modules
- Git
- Package Repo Mirrors (Apt, Yum) 
- Syslog or other logging environment that will collect build logs from hosts
- NTP


#### Post-Configuration

* Once a build server is received from the build master, it will have the aforementioned software and ansible libraries installed.  Foreman will be configured with the defined network and DHCP configurations.  Further configuration will happen via Ansible in the lab.
* Switch may need to be provisioned with network requirements — but we should handle this as part of the deploy-server-build process
* Specific OS Images or firmware packages may need to be uploaded.
* Group vars specific to the project / devices will need to be defined:


## Host Build Process:

#### Stage 0 (PreFlight Configuration):

* If specific OS Images or Firmware images are required, they will need to be uploaded to the build server via FTP.  
* A directory structure will be pre-configured with pre-seeded images for reference.  Attention should be payed to the naming conventions as they will be used in the Ansible group_vars and Foreman build templates.

Ansible group_vars defined (or updated) for project devices
- Disk configuration
- Firmware Versions
- OS Distro / Version / Release
- NIC configuration
- Access credentials

An inventory file will need to be created with the following defined for each host:
- iLO/iDRAC/IPMI access (user/pw/ip)
- serial number
- mac address


#### Stage 1 (Ansible Build):

Run Ansible Playbooks that execute the following main components:

Update foreman build environment via ansible Foreman modules.
- Generate server build profile and build templates based on group_vars and load into Foreman
- Generate any pre/post install scripts that will be used by Foreman during OS deploy
- Load OS Images into foreman

Configure host hardware via ansible redfish and idrac_redfish modules.
- set common auth credentials
- configure disks/raid
- configure NICs
- verify hardware / firmware
- update firmware as needed
- set one-time network-boot

Populate foreman with each host via ansible Foreman module. Each host entry may include:
- Serial #
- MAC Address (for DHCP)
- iLO/iDRAC/IPMI connection details
- Build Profile (Templates, OS) 

Reboot host for Stage 2 (OS Install)


#### Stage 2 (OS Install):

Hosts will be set to 1-time network boot and rebooted at the end of Stage 1
DHCP will already be listening for each host’s MAC to assign an IP and begin Foreman build steps.
The OS installation and as much of the related configuration as possible  should be handled within foreman via pre/post install scripts.

#### Stage 3 (Post-Install Configuration and Testing):

After OS installation is complete, ansible playbooks can be used for any final configuration or testing.
Any testing proceedures will need to be defined by customer and CSA, Playbooks to automate tests will need to be created by CSA



## Installation

### Required Software
[Ansible 2.8]
[The Foreman] Manages TFTP, DHCP and PXE host-build environments
[NGINX] Provides proxy services for The Foreman and serves static HTTP content
[Docker] Container runtime environment used for various services
[TFTP-HPA] Serves files to network-booted hosts
[ISC DHCP Server] Dynamic IP Addressing used for host network booting

### Ansible Modules
[Ansible Foreman Modules] Enables Foreman configuration via Ansible

### Ansible Roles
[foreman-yml]
[Ansible TFTP Role]
[Ansible Docker Role]
[Ansible NGINX Role]
[Ansible TFTP Role]
[Ansible ISC DHCP Server Role]

 * The initial configuration of the Foreman Smart-Proxy is handled using [foreman-yml][].  This role supports configuration of DHCP, deployed via [Ansible ISC DHCP Server Role][] and TFTP, deployed via [Ansible TFTP Role][].
 * Extended configuration of the Foreman server is handled via [Ansible Foreman Modules][] and should be handled via ```group_vars```
 * If you wish to use Postgres or MySQL for the Foreman Database backend, you can deploy it in a container using this [Ansible Docker Role][].  The default database is Postgres
 * Proxied requests to the Foreman server and API are handled by NGINX, which is deployed using this [Ansible NGINX Role][].

### Ansible Configuration

#### Example extra_vars for project deploy

Project cucstomization such as ssh users, provisioning keys, tower projects/job templates/inventories, foreman templates/hostgroups/parameters etc. are defined in the ```customproject_*.yml``` files.

Generally speaking, you will not need to edit the ```*_settings.yml``` files.


##### customproject_common.yml
```yaml

timezone: "America/Denver"
shared_storage: False
www_domain: home.prettybaked.com
provisioning_ssh_key: >
  ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAyVfLHB6up7yWh5bkkLBFFgPbG1E7PjKEjzMAz6eVyjPyp1K5bty4FIWPL8gcIh0Yjf6Z0VBB+4OPvwAf6nKtpmZmcbDETRCw6kAWDRz7C6DCJFDHNmoPfMutC5H3R0ZAlWP7zYs5cVF/DN/il2FYn7S7A8/7aewvLVF6WxiHPXYCJ5MuPKshgsg92PqHfBxTCsmzekyuZ3RncqXDSaY621d/0yMRt1gQlHQPFd/UuosPg0X/JiwMGEfcU4os0ix5Iu4eucUkeMxYgHcZhm19kfl0xjTwMLDPovt42j9Wp1p8TeleVaxrXbZaRPQ+MRGw2bLDWyKjx0GRHIDGv38Uyw==

sshd_permit_root_login: 'yes'

ssh_users:
  ajanis:
    password: "{{ vault_ajanis_pw }}"
    cn: "Alan Janis"
    givenname: "Alan"
    sn: "Janis"
    mail: "alan.janis@gmail.com"
    shell: /bin/zsh
    gecos: ajanis
    uid: 1043
    gid: 1042
    pubkey: >
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC29plb9A6fagU+g+zFX6Bo6SUGj58Z/6nxngV/kd95J0UJc8c3zApz6Q3s6gO1+86rpHoZOKauUnXb/Kcg41ncpluDsLxu1gChY8D8jsAPxG18GxlJuB8d5rgFdX9VnCWNg4w9VuYvwv3n/7hoELyU/oC68to3VIIB4yLMGhuGY3JOqwmpckl1/NhQZCeP+BTXyEib2GjJ4jqbpKXuId8/gp3UpHQ37enLF8eLHH7ue4+TxL1F1uSw3phkqLvhRWPdsYTP4ttfcReqOLpJet87yUiO0xKsUryBuCAngeSUP4hYGJT8i3RbGSvyoXoLJEGIt7hFLP7dI78WPlOUIHHXNml62UKF3c8mFww4qRB95RRWTrILSftkjlHqFSIVxXNtS+AMggc60HT8L8zi5BhHg66PHAo/4JWLmnfgfdMbEqpWxxa21RhTv0LySXs1IOWspXAseNWrxBCrmdxPudMPXSG6f4Gc7STs72MTWQ/0ti6rOsgMvspLshM+wWJ6L0dLRUbti+hHBcdja/6J5MrOMUzgzT+VLmJG4cNP2dAfqseW6t92B9RGP1rICja5PTrIvy0kh6lyVkxnkWlC10WJo2m+VyifPjIPBkekJDfKic9VKe05glMWl8lUillQ0Y9kejFzoLXX8rfbdTmCJYg3qXTrZ4lsZQjr8dGSI4HMqw== ajanis@wwt
    state: present
  wwt:
    password: "{{ vault_wwt_pw }}"
    cn: "WWT Admin"
    givenname: "WWT"
    sn: "Admin"
    mail: "alan.janis@gmail.com"
    shell: /bin/bash
    gecos: wwt
    uid: 1044
    gid: 1042
    pubkey: >
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC29plb9A6fagU+g+zFX6Bo6SUGj58Z/6nxngV/kd95J0UJc8c3zApz6Q3s6gO1+86rpHoZOKauUnXb/Kcg41ncpluDsLxu1gChY8D8jsAPxG18GxlJuB8d5rgFdX9VnCWNg4w9VuYvwv3n/7hoELyU/oC68to3VIIB4yLMGhuGY3JOqwmpckl1/NhQZCeP+BTXyEib2GjJ4jqbpKXuId8/gp3UpHQ37enLF8eLHH7ue4+TxL1F1uSw3phkqLvhRWPdsYTP4ttfcReqOLpJet87yUiO0xKsUryBuCAngeSUP4hYGJT8i3RbGSvyoXoLJEGIt7hFLP7dI78WPlOUIHHXNml62UKF3c8mFww4qRB95RRWTrILSftkjlHqFSIVxXNtS+AMggc60HT8L8zi5BhHg66PHAo/4JWLmnfgfdMbEqpWxxa21RhTv0LySXs1IOWspXAseNWrxBCrmdxPudMPXSG6f4Gc7STs72MTWQ/0ti6rOsgMvspLshM+wWJ6L0dLRUbti+hHBcdja/6J5MrOMUzgzT+VLmJG4cNP2dAfqseW6t92B9RGP1rICja5PTrIvy0kh6lyVkxnkWlC10WJo2m+VyifPjIPBkekJDfKic9VKe05glMWl8lUillQ0Y9kejFzoLXX8rfbdTmCJYg3qXTrZ4lsZQjr8dGSI4HMqw== ajanis@wwt
    state: present

ssh_groups:
  admin:
    description: Administrators Group
    gid: 1042
    members:
      - ajanis
  users:
    description: SSH Users Group
    gid: 2002
    members:
      - ajanis
      - wwt

```
##### customproject_foreman.yml
```yaml

foreman_proxy_dhcp_subnets:
  - name: "10.0.10.0/24"
    network: "10.0.10.0"
    mask: "255.255.255.0"
    gateway: "10.0.10.1"
    from_ip: "10.0.10.240"
    to_ip: "10.0.10.250"
    mtu: 9000
    domains:
    - "{{ www_domain }}"
    dns_primary: 10.0.10.1

foreman_installation_mediums:
  - name: "Ubuntu mirror"
    path: "http://archive.ubuntu.com/ubuntu"
    os_family: Debian

foreman_operatingsystems:
  - name: "Ubuntu"
    description: "Ubuntu 18.04"
    release_name: "bionic"
    family: "Debian"
    major: "18"
    minor: "4"
    provisioning_templates:
      - "Preseed default iPXE"
      - "Preseed default"
      - "Preseed default PXELinux"
      - "Preseed default finish"
      - "Preseed default user data"
    ptables:
      - "Preseed default"
    architectures:
      - x86_64
    media: "Ubuntu mirror"

foreman_os_default_templates:
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "finish"
    provisioning_template: "Preseed default finish"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "provision"
    provisioning_template: "Preseed default"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "PXELinux"
    provisioning_template: "Preseed default PXELinux"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "iPXE"
    provisioning_template: "Preseed default iPXE"
  - operatingsystem: "Ubuntu 18.04"
    template_kind: "user_data"
    provisioning_template: "Preseed default user data"

foreman_hostgroups:
  - name: "kvm"
    description: "KVM Hostgroup"
    architecture: x86_64
    operatingsystem: "Ubuntu 18.04"
    medium: "Ubuntu mirror"
    ptable: "Preseed default"
    domain: "{{ www_domain }}"
    subnet: "10.0.10.0/24"
  - name: "arthur"
    description: "Arthur KVM Hypervisor Hostgroup"
    parent: kvm
    compute_resource: arthur
  - name: "ford"
    description: "Ford KVM Hypervisor Hostgroup"
    parent: kvm
    compute_resource: ford

foreman_compute_resources:
  - name: "arthur"
    provider: libvirt
    provider_params:
      url: qemu+tcp://10.0.10.85/system
      display_type: vnc
  - name: "ford"
    provider: libvirt
    provider_params:
      url: qemu+tcp://10.0.10.87/system
      display_type: vnc

foreman_hosts:
  buildserver05:
    name: buildserver05
    hostgroup: arthur
    provider: kvm
    cpu_cores: 4
    memory: "4G"
    disk_size: "20G"
    ip: 10.0.10.245
  buildserver04:
    name: buildserver04
    hostgroup: arthur
    provider: kvm
    cpu_cores: 4
    memory: "4G"
    disk_size: "20G"
    ip: 10.0.10.244
  buildserver03:
    name: buildserver03
    hostgroup: arthur
    provider: kvm
    cpu_cores: 4
    memory: "4G"
    disk_size: "20G"
    ip: 10.0.10.243
  buildserver02:
    name: buildserver02
    hostgroup: arthur
    provider: kvm
    cpu_cores: 4
    memory: "4G"
    disk_size: "20G"
    ip: 10.0.10.242
  buildserver01:
    name: buildserver01
    hostgroup: ford
    provider: kvm
    cpu_cores: 4
    memory: "4G"
    disk_size: "20G"
    ip: 10.0.10.241

```
##### customproject_tower.yml
```yaml
ansible_hostbuild_project: git@github.com:ajanis/wwt-build-pods.git
awx_notification_email: 9412863602@mypixmessages.com
awx_host_config_key: 16a4a143-0f6e-49b6-8051-fe135b6fdffa

tower_inventories:
  - name: "build-servers"
    description: "Intel NUC Build Servers"
    organization: "Default"
  - name: "build-masters"
    description: "Build Master"
    organization: "Default"
    extra_vars_path: "extra_vars/*.yml"

tower_hosts:
  - name: "{{ ansible_fqdn }}"
    description: "Build-Master"
    inventory: "build-masters"
    variables:
      ansible_ssh_host: "{{ ansible_default_ipv4.address }}"
      ansible_python_interpreter: "/usr/bin/env python3"
  - name: "buildserver01"
    description: "Build-Server {{ ansible_domain }}"
    inventory: "build-servers"
    variables:
      ansible_ssh_host: "{{ foreman_hosts['buildserver01']['ip'] }}"
      ansible_python_interpreter: "/usr/bin/env python3"
  - name: "buildserver02"
    description: "Build-Server {{ ansible_domain }}"
    inventory: "build-servers"
    variables:
      ansible_ssh_host: "{{ foreman_hosts['buildserver02']['ip'] }}"
      ansible_python_interpreter: "/usr/bin/env python3"
  - name: "buildserver03"
    description: "Build-Server {{ ansible_domain }}"
    inventory: "build-servers"
    variables:
      ansible_ssh_host: "{{ foreman_hosts['buildserver03']['ip'] }}"
      ansible_python_interpreter: "/usr/bin/env python3"
  - name: "buildserver04"
    description: "Build-Server {{ ansible_domain }}"
    inventory: "build-servers"
    variables:
      ansible_ssh_host: "{{ foreman_hosts['buildserver04']['ip'] }}"
      ansible_python_interpreter: "/usr/bin/env python3"
  - name: "buildserver05"
    description: "Build-Server {{ ansible_domain }}"
    inventory: "build-servers"
    variables:
      ansible_ssh_host: "{{ foreman_hosts['buildserver05']['ip'] }}"
      ansible_python_interpreter: "/usr/bin/env python3"

tower_credentials:
  - name: ansible-root
    kind: ssh
    ssh_key_data: "{{ github_deploy_key }}"
  - name: ansible-github
    kind: scm
    ssh_key_data: "{{ github_deploy_key }}"
  - name: ansible-vault
    vault_id: ansible-vault
    kind: vault
    vault_password: "{{ vault_awx_vault_password }}"

tower_projects:
  - name: "hostbuild"
    description: "WWT Host-Build Project"
    scm_type: git
    scm_credential: ansible-github
    scm_clean: yes
    scm_branch: master
    scm_update_on_launch: yes
    scm_url: "{{ ansible_hostbuild_project }}"
    local_path: "hostbuild"

tower_job_templates:
  - name: "Reimage Host buildserver01"
    job_type: "run"
    fact_caching_enabled: yes
    diff_mode_enabled: yes
    concurrent_jobs_enabled: yes
    inventory: "build-masters"
    project: "hostbuild"
    playbook: "build-host-deploy.yml"
    extra_vars_path: "host_vars/buildserver01.yml"
    vault_credential: "ansible-vault"
  - name: "Reimage Host buildserver02"
    job_type: "run"
    fact_caching_enabled: yes
    diff_mode_enabled: yes
    concurrent_jobs_enabled: yes
    inventory: "build-masters"
    project: "hostbuild"
    playbook: "build-host-deploy.yml"
    extra_vars_path: "host_vars/buildserver02.yml"
    vault_credential: "ansible-vault"
  - name: "Reimage Host buildserver03"
    job_type: "run"
    fact_caching_enabled: yes
    diff_mode_enabled: yes
    concurrent_jobs_enabled: yes
    inventory: "build-masters"
    project: "hostbuild"
    playbook: "build-host-deploy.yml"
    extra_vars_path: "host_vars/buildserver03.yml"
    vault_credential: "ansible-vault"
  - name: "Reimage Host buildserver04"
    job_type: "run"
    fact_caching_enabled: yes
    diff_mode_enabled: yes
    concurrent_jobs_enabled: yes
    inventory: "build-masters"
    project: "hostbuild"
    playbook: "build-host-deploy.yml"
    extra_vars_path: "host_vars/buildserver04.yml"
    vault_credential: "ansible-vault"
  - name: "Reimage Host buildserver05"
    job_type: "run"
    fact_caching_enabled: yes
    diff_mode_enabled: yes
    concurrent_jobs_enabled: yes
    inventory: "build-masters"
    project: "hostbuild"
    playbook: "build-host-deploy.yml"
    extra_vars_path: "host_vars/buildserver05.yml"
    vault_credential: "ansible-vault"
  - name: "Host Configure"
    job_type: "run"
    host_config_key: "{{ awx_host_config_key }}"
    fact_caching_enabled: yes
    concurrent_jobs_enabled: yes
    diff_mode_enabled: yes
    inventory: "build-servers"
    ask_limit: no
    project: "hostbuild"
    playbook: "build-host-configure.yml"
    credential: "ansible-root"
    vault_credential: "ansible-vault"
```
##### docker_settings.yml
```yaml
docker_networks:
  docker_awx:
    ipam_config:
      - subnet: 172.3.27.0/24

docker_containers:

  postgres:
    description: "Foreman Postgresql DB"
    image: postgres:11
    env:
      PUID: '0'
      PGID: '0'
      VERSION: 'latest'
      POSTGRES_USER: '{{ foreman_db_username }}'
      POSTGRES_PASSWORD: '{{ vault_foreman_db_password }}'
      POSTGRES_DB: '{{ foreman_db_name }}'
    ports:
      - '5432:5432'
    volumes:
      - '/etc/localtime:/etc/localtime:ro'
      - '/var/lib/postgresql/data:/var/lib/postgresql/data'

  awx_postgres:
    description: AWX Postgres DB
    image: postgres:10
    hostname: postgres
    networks:
      - name: docker_awx
        aliases: postgres
    volumes:
      - "{{ awx_postgres_data_dir }}/10/data/:/var/lib/postgresql/data/pgdata:Z"
    env:
      POSTGRES_USER: "{{ awx_pg_username }}"
      POSTGRES_PASSWORD: "{{ awx_pg_password }}"
      POSTGRES_DB: "{{ awx_pg_database }}"
      PGDATA: "/var/lib/postgresql/data/pgdata"
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awx_rabbitmq:
    description: AWX RabbitMQ Service
    image: "{{ awx_rabbitmq_image }}"
    volumes:
      - '/etc/localtime:/etc/localtime:ro'
    networks:
      - name: docker_awx
        aliases: rabbitmq
    hostname: rabbitmq
    env:
      RABBITMQ_DEFAULT_VHOST: "{{ awx_rabbitmq_default_vhost }}"
      RABBITMQ_DEFAULT_USER: "{{ awx_rabbitmq_user }}"
      RABBITMQ_DEFAULT_PASS: "{{ awx_rabbitmq_password | quote }}"
      RABBITMQ_ERLANG_COOKIE: "{{ awx_rabbitmq_erlang_cookie }}"
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awx_memcached:
    description: AWX Memcached Service
    image: "{{ awx_memcached_image }}:{{ awx_memcached_version }}"
    hostname: "{{ awx_memcached_host }}"
    networks:
      - name: docker_awx
        aliases: "{{ awx_memcached_host }}"
    volumes:
      - '/etc/localtime:/etc/localtime:ro'
    env:
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awx:
    description: AWX Service
    image: "{{ awx_dockerhub_base }}/awx_task:{{ awx_dockerhub_version }}"
    hostname: "{{ awx_task_hostname }}"
    depends_on:
      - docker-awx_memcached.service
      - docker-awx_rabbitmq.service
      - docker-awx_postgres.service
    networks:
      - name: docker_awx
        aliases:
          - task
          - awx
        links:
          - awxweb:awx_web
          - awxweb:web
    user: root
    volumes:
      - "{{ awx_docker_compose_dir }}/SECRET_KEY:/etc/tower/SECRET_KEY"
      - "{{ awx_docker_compose_dir }}/environment.sh:/etc/tower/conf.d/environment.sh"
      - "{{ awx_docker_compose_dir }}/credentials.py:/etc/tower/conf.d/credentials.py"
      - "{{ awx_project_data_dir }}:/var/lib/awx/projects:rw"
    env:
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"

  awxweb:
    description: AWX Web Application
    image: "{{ awx_dockerhub_base }}/awx_web:{{ awx_dockerhub_version }}"
    depends_on:
      - docker-awx_memcached.service
      - docker-awx_rabbitmq.service
      - docker-awx_postgres.service
    ports:
      - "{{ awx_host_port }}:8052"
    networks:
      - name: docker_awx
        aliases:
          - web
          - awxweb
        links:
          - awx:awx_task
          - awx:task
    hostname: "{{ awx_web_hostname }}"
    user: root
    volumes:
      - "{{ awx_docker_compose_dir }}/SECRET_KEY:/etc/tower/SECRET_KEY"
      - "{{ awx_docker_compose_dir }}/environment.sh:/etc/tower/conf.d/environment.sh"
      - "{{ awx_docker_compose_dir }}/credentials.py:/etc/tower/conf.d/credentials.py"
      - "{{ awx_docker_compose_dir }}/nginx.conf:/etc/nginx/nginx.conf:ro"
      - "{{ awx_project_data_dir }}:/var/lib/awx/projects:rw"
    env:
      http_proxy: "{{ http_proxy | default('') }}"
      https_proxy: "{{ https_proxy | default('') }}"
      no_proxy: "{{ no_proxy | default('') }}"
```
##### foreman_settings.yml
```yaml

foreman_install_version: 1.24
foreman_plugins_version: 1.24

foreman_hostname: foreman
foreman_interface: 0.0.0.0
foreman_port: 3000
foreman_proxy_tftp: True
foreman_proxy_dhcp: True
foreman_db_name: foreman
foreman_db_username: foreman
foreman_db_password: "{{ vault_foreman_db_password }}"
foreman_admin_user: admin
foreman_admin_password: "{{ vault_foreman_admin_password }}"
foreman_env: production

foreman_extra_settings:
  - name: "root_pass"
    value: "{{ vault_foreman_admin_password }}"

foreman_global_parameters:
  - name: remote_execution_ssh_keys
    value: "{{ provisioning_ssh_key }}"
  - name: ansible_host_config_key
    value: "{{ awx_host_config_key }}"
  - name: ansible_tower_fqdn
    value: "{{ awx_web_url }}"
  - name: ansible_enabled
    parameter_type: boolean
    value: "True"
  - name: ansible_tower_provisioning
    parameter_type: boolean
    value: "True"
  - name: additional-packages
    parameter_type: string
    value: 'curl'
  - name: http-proxy
    parameter_type: string
    value: "{{ foreman_proxy_bind_host }}"
  - name: http-proxy-port
    parameter_type: integer
    value: 3142
```
##### nginx_settings.yml
```yaml

nginx_backends:
  - service: foreman
    servers:
      - localhost:3000

nginx_vhosts:
  - servername: "{{ foreman_hostname | default(ansible_hostname) + '.' + www_domain | default(ansible_domain) }}"
    serveralias: "{{ ansible_default_ipv4.address }} {{ ansible_fqdn }}"
    serverlisten: 80
    extra_parameters: |
      error_page 404 /404.html;
      error_page 500 502 503 504 /50x.html;
    locations:
      - name: /
        proxy: True
        backend: foreman
      - name: 404.html
        docroot: "/usr/share/nginx/html"
      - name: 50x.html
        docroot: "/usr/share/nginx/html"

nginx_vhosts_ssl: []
```
##### tower_settings.yml
```yaml
awx_host_port: 8080
awx_web_url: "http://{{ ansible_default_ipv4.address }}:{{ awx_host_port }}"
awx_project_data_dir: /var/lib/awx

awx_rabbitmq_erlang_cookie: "{{ vault_rabbitmq_erlang_cookie }}"
awx_rabbitmq_password: "{{ vault_rabbitmq_password }}"

awx_pg_password: "{{ vault_pg_password }}"
awx_pg_admin_password: "{{ vault_pg_admin_password }}"

awx_secret_key: "{{ vault_secret_key }}"

awx_admin_user: admin
awx_admin_password: "{{ vault_awx_admin_password }}"
```



#### Example Playbook
```yaml
---
- name: "Deploy Foreman Server"
  hosts: all
  remote_user: root
  gather_facts: yes
  vars_files:
    - vault.yml

  tasks:

    - include_vars:
        dir: extra_vars

    - name: Wait for server to come online
      wait_for_connection:
        delay: 60
        sleep: 10
        connect_timeout: 5
        timeout: 300

    - import_role:
        name: common

    - import_role:
        name: isc_dhcp_server
      when: foreman_proxy_dhcp

    - import_role:
        name: tftp
      when: foreman_proxy_tftp

    - import_role:
        name: nginx

    - import_role:
        name: awx

    - import_role:
        name: docker

    - import_role:
        name: awx
        tasks_from: container-tasks.yml

    - import_role:
        name: foreman

    - import_role:
        name: ansible-project

```

## License

MIT

## Author Information

Created by Alan Janis

[Ansible Foreman Modules]: https://github.com/theforeman/foreman-ansible-modules
[Ansible Docker Role]: https://github.com/ajanis/ansible-docker.git
[foreman-yml]: https://pypi.org/project/foreman-yml
[ansible tftp role]:  https://github.com/ajanis/ansible-tftp.git
[ansible nginx role]:  https://github.com/ajanis/ansible-nginx.git
[itc automated build pods project]:  https://github.com/ajanis/itc-build-pods.git
[ansible isc dhcp server role]:  https://github.com/ajanis/ansible-isc-dhcp-server.git
[tftp-hpa]: https://help.ubuntu.com/community/TFTP
[docker]: https://www.docker.com/
[the foreman]: https://www.theforeman.org/
[ansible]: https://www.ansible.com/
[nginx]: https://www.nginx.com/
[isc dhcp server]: https://www.nginx.com/
