# Ansible Nagios4 playbook

## Table of Contents

<!--toc:start-->

- [Ansible Nagios4 playbook](#ansible-nagios4-playbook)
  - [Table of Contents](#table-of-contents)
  - [Apache Basic auth](#apache-basic-auth)
  - [Nagios installation role using Ansible Vault](#nagios-installation-role-using-ansible-vault)
  - [Ansible Vault](#ansible-vault)
  - [Ansible Vault parameters for htdigest.users](#ansible-vault-parameters-for-htdigestusers)
  - [Playbook run using Ansible Vault](#playbook-run-using-ansible-vault)
  - [Alternative to Ansible Vault](#alternative-to-ansible-vault)
  <!--toc:end-->

## Apache Basic auth

The htdigest.users password can be generated through Ansible or generated on the Nagios server and copied as a template.

## Nagios installation role using Ansible Vault

In order to use Ansible Vault, Python's passlib is needed on the Nagios host to decrypt the vault.

```yml
---

- name: Install Nagios packages
  ansible.builtin.package:
  name: "{{ item }}"
  state: present
  update_cache: true
  with_items:
  - nagios4
  - nagios4-common
  - monitoring-plugins-contrib
  - nagios-nrpe-plugin
  - python3-passlib
    tags:
  - installation
```

## Ansible Vault

```bash
ansible-vault encrypt inventory/group_vars/monitoring.yml
ansible-vault view inventory/group_vars/monitoring.yml
```

```txt
nagiosadmin: nagiosadmin
nagiospassword: p4ssw0rd
```

## Ansible Vault parameters for htdigest.users

```yml
- name: Set htpasswd for Nagios admin
  ansible.builtin.htpasswd:
  path: /etc/nagios4/htdigest.users
  name: "{{ nagiosadmin }}"
  password: "{{ nagiospassword }}"
  crypt_scheme: sha256_crypt
  owner: nagios
  group: www-data
  mode: '0640'
  become: yes
```

## Playbook run using Ansible Vault

In the case of an Ansible Vault scenario, enter the vault password to run a playbook.

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass
ansible-playbook playbooks/nagios_configuration.yml --ask-vault-pass
```

## Alternative to Ansible Vault

The alternative is to generate the digest on the Nagios host and copy it to the Ansible host as a template.
This way the playbooks can be executed without providing a password and as such, can be scheduled easier.

The teemplate provided uses nagiosadmin / p4ssw0rd. It's advised to replace this placeholder file with a stronger password.

```bash
sudo htpasswd /etc/nagios4/htdigest.users nagiosadmin
```
