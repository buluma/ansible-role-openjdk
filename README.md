# [Ansible role openjdk](#ansible-role-openjdk)

Ansible Role for OpenJDK Installation.

|GitHub|GitLab|Downloads|Version|
|------|------|---------|-------|
|[![github](https://github.com/buluma/ansible-role-openjdk/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-openjdk/actions)|[![gitlab](https://gitlab.com/shadowwalker/ansible-role-openjdk/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-openjdk)|[![downloads](https://img.shields.io/ansible/role/d/buluma/openjdk)](https://galaxy.ansible.com/buluma/openjdk)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-openjdk.svg)](https://github.com/buluma/ansible-role-openjdk/releases/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-openjdk/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---

- hosts: all
  remote_user: root
  become: true

  roles:
    - role: buluma.bootstrap
    - role: buluma.maven
    - role: buluma.openjdk
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-openjdk/blob/master/molecule/default/prepare.yml):

```yaml
---
- hosts: all
  remote_user: root
  become: true
  gather_facts: false

  roles:
    - role: buluma.bootstrap

  tasks:
    - name: Redhat | subscription-manager register
      ansible.builtin.raw: |
        set -eu
        subscription-manager register \
        --username={{ lookup('env', 'REDHAT_USERNAME') }} \
        --password={{ lookup('env', 'REDHAT_PASSWORD') }} \
        --auto-attach
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Debian | apt-get install python3
      ansible.builtin.raw: |
        set -eu
        apt-get update
        DEBIAN_FRONTEND=noninteractive apt-get install -y python3
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Redhat | yum install python3
      ansible.builtin.raw: |
        set -eu
        yum makecache
        yum install -y python3
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Suse | zypper install python3
      ansible.builtin.raw: |
        set -eu
        zypper -n --gpg-auto-import-keys refresh
        zypper -n install -y python3
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

- name: prepare | all
  hosts: all
  remote_user: root
  become: true
  tasks:
    - name: Cp -rfT /etc/skel /root
      ansible.builtin.raw: |
        set -eu
        cp -rfT /etc/skel /root
        rm -rf /root/.bash_logout
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Setenforce 0
      ansible.builtin.raw: |
        set -eu
        setenforce 0
        sed -i 's/^SELINUX=.*$/SELINUX=permissive/g' /etc/selinux/config
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Systemctl stop iptables.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop iptables.service
        systemctl disable iptables.service
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Systemctl stop firewalld.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop firewalld.service
        systemctl disable firewalld.service
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Systemctl stop ufw.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop ufw.service
        systemctl disable ufw.service
      args:
        executable: "/bin/bash"
      changed_when: false
      failed_when: false

    - name: Debian | apt-get install *.deb
      ansible.builtin.raw: |
        set -eu
        DEBIAN_FRONTEND=noninteractive apt-get install -y bzip2 ca-certificates curl gcc gnupg gzip hostname iproute2 passwd procps python3 python3-apt python3-jmespath python3-lxml python3-pip python3-setuptools python3-venv python3-virtualenv python3-wheel rsync sudo tar unzip util-linux zip
      args:
        executable: "/bin/bash"
      when: ansible_os_family | lower == "debian"
      changed_when: false
      failed_when: false

    # TODO: Pinning due to CVE-2024-3094
    - name: Debian | Try to install xz-utils packages
      ansible.builtin.pip:
        name: xz-utils==5.4.6
        state: present
      when: ansible_os_family | lower == "debian"
      changed_when: false
      failed_when: false

    - name: Fedora | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      args:
        executable: "/bin/bash"
      when: ansible_distribution | lower == "fedora"
      changed_when: false
      failed_when: false

    - name: Redhat-9 | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum-config-manager --enable crb || echo $?
        yum-config-manager --enable codeready-builder-for-rhel-9-x86_64-rpms || echo $?
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      args:
        executable: "/bin/bash"
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "9"
      changed_when: false
      failed_when: false

    - name: Redhat-8 | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      args:
        executable: "/bin/bash"
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "8"
      changed_when: false
      failed_when: false

    - name: Redhat-7 | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        subscription-manager repos --enable=rhel-7-server-optional-rpms || echo $?
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-plugin-versionlock yum-utils zip
      args:
        executable: "/bin/bash"
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "7"
      changed_when: false
      failed_when: false

    - name: Suse | zypper -n install *.rpm
      ansible.builtin.raw: |
        set -eu
        zypper -n install -y bzip2 ca-certificates curl gcc gpg2 gzip hostname iproute2 procps python3 python3-jmespath python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow sudo tar unzip util-linux xz zip
      args:
        executable: "/bin/bash"
      when: ansible_os_family | lower == "suse"
      changed_when: false
      failed_when: false
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-openjdk/blob/master/defaults/main.yml):

```yaml
---

# OpenJDK release.
openjdk_release: "17"

# Specify JAVA_HOME.
openjdk_home: "{{ _openjdk_home[openjdk_release] }}"
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-openjdk/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Build Status GitHub](https://github.com/buluma/ansible-role-bootstrap/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-bootstrap/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-bootstrap)|
|[buluma.maven](https://galaxy.ansible.com/buluma/maven)|[![Build Status GitHub](https://github.com/buluma/ansible-role-maven/workflows/Ansible%20Molecule/badge.svg)](https://github.com/buluma/ansible-role-maven/actions)|[![Build Status GitLab](https://gitlab.com/shadowwalker/ansible-role-maven/badges/master/pipeline.svg)](https://gitlab.com/shadowwalker/ansible-role-maven)|

## [Context](#context)

This role is part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-openjdk/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[Ubuntu](https://hub.docker.com/r/buluma/ubuntu)|all|
|[EL](https://hub.docker.com/r/buluma/enterpriselinux)|all|
|[opensuse](https://hub.docker.com/r/buluma/opensuse)|all|
|[Debian](https://hub.docker.com/r/buluma/debian)|all|
|[Fedora](https://hub.docker.com/r/buluma/fedora)|all|

The minimum version of Ansible required is 4.10, tests have been done on:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them on [GitHub](https://github.com/buluma/ansible-role-openjdk/issues).

## [License](#license)

[Apache-2.0](https://github.com/buluma/ansible-role-openjdk/blob/master/LICENSE).

## [Author Information](#author-information)

[Michael Buluma](https://buluma.github.io/)

