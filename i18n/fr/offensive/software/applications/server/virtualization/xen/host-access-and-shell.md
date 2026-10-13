---
order: 1
title: "Accès à l'hôte et shell : exécution dans le dom0 Xen"
description: "Le domaine de contrôle de Xen, dom0, est un domaine Linux (ou autre) privilégié qui exécute la toolstack et, en général, les backends de périphériques. Obtenir un shell dans dom0, via une évasion d'invité, la toolstack ou SSH, équivaut au contrôle de l'hôte : il administre chaque domaine, accède à tous les disques des invités et peut injecter dans les invités ou les recréer via les outils Xen."
keywords:
  - dom0
  - toolstack xen
  - xl
  - xenstore
  - accès à l'hôte
---

# Accès à l'hôte et shell

dom0 est le domaine de contrôle de Xen : un OS privilégié (couramment Linux) qui démarre en premier, exécute la toolstack (`xl`/libxl, ou XAPI sur XenServer/XCP-ng), héberge les backends de périphériques et administre tous les autres domaines. Obtenir un shell dans dom0 équivaut au contrôle de l'hôte. Cela provient d'une évasion invité-vers-hôte qui aboutit dans dom0 (via un backend) ou dans l'hyperviseur, d'un accès à la toolstack ou à XAPI, ou de SSH avec des identifiants dom0. Depuis dom0, un attaquant contrôle chaque domaine via les outils, lit tous les disques des invités, manipule la base de configuration XenStore et persiste sur l'hôte.

## Atteindre et utiliser dom0

```bash
# SSH into dom0, or arrive via a backend/hypervisor escape
xl list                                   # all domains
xl console <domid>                        # attach to a guest console
# XenStore holds domain configuration and PV device wiring
xenstore-ls /local/domain                 # per-domain config visible from dom0
# run against guest disks (see disk theft) and recreate/modify domains
xl create /etc/xen/<guest>.cfg
```

## Ce que dom0 apporte

```text
dom0 control includes:
- lifecycle control of every domain (create, destroy, pause, console)
- access to all guest virtual disks through the backends/storage (offline theft)
- the XenStore database wiring devices and config for all domains
- on XenServer/XCP-ng, the XAPI management plane and pool-wide control
Persistence is standard for the dom0 OS (systemd/cron/keys), plus domain config under /etc/xen.
```

## Notes d'exploitation

- dom0 est le domaine privilégié, de sorte que root dans dom0 équivaut au contrôle de niveau hôte de tout le système Xen, équivalant à compromettre l'hôte hyperviseur.
- La toolstack (`xl`) et XenStore sont les surfaces de contrôle depuis dom0 ; XenStore expose et configure le câblage des périphériques de chaque domaine, utile à la fois pour l'énumération et le sabotage.
- Sur XenServer/XCP-ng, dom0 exécute aussi XAPI et participe à un pool de ressources, de sorte que l'accès à dom0 s'étend à l'échelle du pool ; voir [Plan de gestion](management-plane.md).
- La persistance suit l'OS de dom0 ; les fichiers de configuration de domaine sous `/etc/xen` sont un emplacement supplémentaire, spécifique à la virtualisation.

## Références

- [Toolstack Xen (xl/libxl)](https://xenbits.xen.org/docs/)
- [Documentation XenStore](https://wiki.xenproject.org/wiki/XenStore)
