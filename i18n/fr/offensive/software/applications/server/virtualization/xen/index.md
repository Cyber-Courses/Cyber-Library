---
order: 4
title: "Xen : attaquer l'hyperviseur de paravirtualisation bare-metal"
description: "Xen est un hyperviseur de type 1 où un dom0 privilégié héberge la pile de contrôle et les pilotes backend pour les I/O des invités. L'évasion invité-vers-hôte vise l'interface des hypercalls, les grant tables et les event channels, et les pilotes backend paravirtuels tournant dans dom0, en parallèle du vol des disques des invités et du plan de gestion de la toolstack."
keywords:
  - xen
  - dom0
  - hypercall
  - grant tables
  - paravirtuel
---

# Xen

Xen est un hyperviseur de type 1 : un hyperviseur mince tourne sur le matériel, et un domaine de contrôle privilégié (dom0) héberge la toolstack et, pour la plupart des configurations, les moitiés backend des périphériques paravirtuels des invités. Les invités (domU) atteignent les fonctionnalités de l'hôte via des hypercalls vers l'hyperviseur et via le modèle de pilote paravirtuel scindé, où un frontend dans l'invité dialogue avec un backend dans dom0 (ou un domaine pilote) en utilisant la mémoire partagée via les grant tables et les notifications via les event channels. La surface d'évasion est donc l'interface des hypercalls, la machinerie des grant tables et des event channels, et les pilotes backend, plus le vol habituel de disque d'invité et la toolstack de gestion.

```bash
# in a Xen guest
dmesg | grep -i xen; ls /sys/bus/xen* 2>/dev/null
cat /sys/hypervisor/type 2>/dev/null       # "xen"
# in dom0: the toolstack and domains
xl list 2>/dev/null
```

## Sous-thèmes

- **[Évasion invité-vers-hôte](guest-to-host-escape/index.md)** : hypercalls, grant tables et event channels, et pilotes backend.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : exécution dans dom0.
- **[Plan de gestion](management-plane.md)** : la toolstack Xen et XAPI.
- **[Vol de disque et de snapshot](disk-and-snapshot-theft.md)** : s'emparer des disques virtuels des invités.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs récurrents d'hypercall et de backend.

## Références

- [Documentation du projet Xen](https://xenbits.xen.org/docs/)
- [Avis de sécurité Xen (XSA)](https://xenbits.xen.org/xsa/)
- [Architecture de Xen](https://wiki.xenproject.org/wiki/Xen_Project_Software_Overview)
