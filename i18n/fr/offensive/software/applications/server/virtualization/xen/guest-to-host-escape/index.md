---
order: 2
title: "Évasion invité-vers-hôte : sortir d'un domU Xen"
description: "Un invité Xen s'échappe vers l'hyperviseur ou le dom0 privilégié par trois surfaces : l'interface des hypercalls que l'invité appelle vers l'hyperviseur, les mécanismes de grant tables et d'event channels qui partagent mémoire et signaux entre domaines, et les pilotes backend paravirtuels de dom0 qui analysent les requêtes en anneau de l'invité. Chacune est atteignable par l'invité et a produit des évasions."
keywords:
  - évasion xen
  - hypercall
  - grant tables
  - event channels
  - pv backend
---

# Évasion invité-vers-hôte

Un domU Xen atteint l'extérieur par des interfaces bien définies, et chacune est une surface d'évasion. Les hypercalls sont les appels directs de l'invité vers l'hyperviseur, de sorte qu'une faille dans la gestion des hypercalls corrompt ou subvertit l'hyperviseur lui-même. Les grant tables et les event channels sont la manière dont les domaines partagent des pages mémoire et se signalent mutuellement ; une mauvaise gestion y crée un pont d'un invité vers la mémoire de dom0 ou de l'hyperviseur. Et les pilotes backend paravirtuels (bloc, réseau) tournant dans dom0 analysent les requêtes en anneau qu'un frontend invité publie, de sorte qu'un bug de backend donne l'exécution de code dans le dom0 privilégié. La cible est celle de ces surfaces que la configuration expose ; les invités PV, PVH et HVM atteignent des combinaisons quelque peu différentes.

```bash
# the PV interfaces visible in a Xen guest
ls /sys/bus/xen-backend 2>/dev/null; ls /dev/xen 2>/dev/null
cat /proc/xen/capabilities 2>/dev/null
dmesg | grep -iE 'grant|event channel|xenbus'
```

## Sous-thèmes

- **[Hypercalls](hypercalls.md)** : l'interface d'appel invité-vers-hyperviseur.
- **[Grant tables et event channels](grant-tables-and-event-channels.md)** : la machinerie de mémoire et de signaux inter-domaines.
- **[Pilotes backend PV](pv-backend-drivers.md)** : les moitiés backend dom0 des périphériques paravirtuels.

## Références

- [Interface des hypercalls Xen](https://xenbits.xen.org/docs/)
- [Avis de sécurité Xen (XSA)](https://xenbits.xen.org/xsa/)
