---
order: 3
title: "Plan de gestion : l'API Proxmox, l'interface web et le cluster"
description: "Proxmox est administré via une API REST et une interface web sur le port 8006, authentifiées avec des tickets ou des tokens d'API et autorisées par un système de rôles. L'accès contrôle chaque VM et conteneur : console et exécution de commandes via le guest agent, opérations de disque et de sauvegarde, et shell du nœud. Des failles de ticket, de token ou d'élévation de privilèges dans cette surface donnent le contrôle du parc sans évasion depuis l'invité."
keywords:
  - proxmox api
  - pveproxy
  - api token
  - ticket
  - pvesh
---

# Plan de gestion

La surface de contrôle de Proxmox est l'API REST et l'interface web servies par `pveproxy` sur le port 8006, avec la CLI `pvesh` comme client local. L'authentification utilise un ticket de connexion (un token signé dans un cookie) ou un token d'API, et l'autorisation est un système de rôles/permissions sur une hiérarchie de chemins. Atteindre cette surface avec des droits suffisants équivaut au contrôle du parc : ouvrir une console de VM, exécuter des commandes via le guest agent, entrer dans des conteneurs, créer et restaurer des sauvegardes, attacher des disques, et ouvrir un shell de nœud. Des failles dans la gestion des tickets ou des tokens, ou une élévation de privilèges au sein du système de rôles, donnent ce contrôle sans aucune évasion depuis l'invité.

## Piloter l'API

```bash
# authenticate for a ticket, or use an API token directly
curl -sk -d 'username=root@pam&password=<pw>' https://<node>:8006/api2/json/access/ticket
# API token form (no login needed)
H='Authorization: PVEAPIToken=user@realm!tokenid=<secret>'
curl -sk -H "$H" https://<node>:8006/api2/json/cluster/resources   # all VMs/cts/nodes
# act: run a command inside a VM via the guest agent
curl -sk -H "$H" -XPOST https://<node>:8006/api2/json/nodes/<node>/qemu/<vmid>/agent/exec \
  -d 'command=id'
# or enter a container / open a node shell through the API
```

## Actions de contrôle

```text
With management access an attacker can:
- execute in VMs via the guest agent, and in containers via pct/exec endpoints
- create/restore vzdump backups and attach/clone disks (data access, see disk theft)
- open the node's shell (termproxy/vncshell) for direct root on the host
- read cluster config, users, tokens, and storage definitions
Permissions are path-scoped, so a limited token still enables whatever its role allows.
```

## Notes d'exploitation

- Les tokens d'API sont des identifiants porteurs limités par rôle ; un token trouvé (dans des scripts, du CI ou de la config) agit directement avec ses permissions, sans mot de passe, alors recherchez les chaînes `PVEAPIToken`.
- L'endpoint exec du guest agent exécute des commandes dans une VM depuis le plan de gestion, et les endpoints termproxy/vncshell ouvrent un shell root de nœud, si bien que l'accès de gestion se convertit en exécution sur l'hôte et l'invité.
- Énumérez les attributions de rôles : un utilisateur ou token non-root avec des privilèges de VM ou de nœud peut quand même permettre un accès console, une restauration de sauvegarde ou un attachement de disque qui atteignent les données ; `access/permissions` montre la portée.
- Identifiez la version Proxmox pour tout avis de sécurité sur un ticket/token ou une élévation de privilèges dans la pile de gestion.

## Références

- [Visionneuse d'API Proxmox VE](https://pve.proxmox.com/pve-docs/api-viewer/)
- [Proxmox VE : gestion des utilisateurs et permissions](https://pve.proxmox.com/pve-docs/chapter-pveum.html)
