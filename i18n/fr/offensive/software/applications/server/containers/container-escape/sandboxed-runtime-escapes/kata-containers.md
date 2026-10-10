---
title: "Kata Containers : s'évader de la VM invitée pour atteindre l'hôte"
description: "Kata Containers exécute chaque conteneur à l'intérieur d'une machine virtuelle légère, de sorte que le conteneur est séparé de l'hôte par un hyperviseur. S'évader nécessite un chemin invité vers hôte : une vulnérabilité de l'hyperviseur ou du modèle de périphérique qui permet de sortir de la VM, ou un abus de l'agent Kata et des canaux virtio qui relient l'invité au runtime côté hôte."
keywords:
  - kata containers
  - évasion de vm
  - kata agent
  - virtio
  - évasion de bac à sable
---

# Kata Containers

Kata Containers enveloppe chaque conteneur (ou pod) dans sa propre machine virtuelle légère dotée d'un noyau invité dédié, exécutée sous un hyperviseur tel que QEMU ou un VMM plus léger. Un processus de conteneur qui s'évaderait d'un conteneur normal n'atteint que le noyau invité, lequel est isolé de l'hôte par la frontière de virtualisation. Sortir nécessite donc le même type de chemin invité vers hôte que n'importe quelle évasion de VM, auquel s'ajoute le plan de contrôle propre à Kata : l'agent Kata à l'intérieur de l'invité, que le runtime côté hôte pilote via un canal série virtio ou vsock pour gérer le conteneur.

Détectez Kata et inspectez l'invité :

```bash
uname -a                                       # a minimal Kata guest kernel
ls /dev | grep -iE 'vport|vsock|kata'          # virtio control channels
dmesg 2>/dev/null | grep -iE 'kata|virtio'
```

## Surfaces d'évasion

- **Hyperviseur et modèle de périphérique** : un bug de sûreté mémoire dans les périphériques émulés du VMM (réseau, bloc, ou un périphérique paravirtualisé) permet au code invité de corrompre la mémoire du VMM côté hôte et de s'exécuter sur l'hôte. Il s'agit d'une évasion de VM standard qui recoupe les évasions du modèle de périphérique de l'hyperviseur documentées pour le VMM sous-jacent.
- **Agent Kata et canaux de contrôle** : l'agent s'exécute dans l'invité et expose une API que le runtime de l'hôte appelle via vsock ou un port série virtio. Une faille dans l'agent, ou la capacité d'atteindre le canal de l'agent depuis la charge de travail du conteneur, peut permettre au conteneur de piloter des opérations de l'agent qu'il ne devrait pas, influençant ce qui s'exécute dans l'invité ou la manière dont l'hôte interagit avec lui.
- **Ressources partagées** : les fonctionnalités qui percent des trous dans la frontière de la VM pour des raisons de performance (systèmes de fichiers partagés via virtio-fs, assignation directe de périphériques) élargissent la surface ; un bug dans le démon de système de fichiers partagé côté hôte peut exposer des fichiers de l'hôte.

## Notes d'exploitation

- La véritable frontière est l'hyperviseur, de sorte qu'une évasion de Kata est fondamentalement une évasion de VM ; les cibles productives sont les périphériques émulés du VMM et tout démon de système de fichiers partagé s'exécutant sur l'hôte.
- Atteindre le canal de l'agent Kata depuis l'intérieur du conteneur est une condition préalable à la voie de l'agent ; par conception, la charge de travail n'est pas censée dialoguer directement avec l'agent, de sorte que cela dépend de l'exposition du canal.
- Comme chaque conteneur est sa propre VM, une évasion réussie de l'hôte depuis un invité Kata ne compromet pas automatiquement les autres invités, contrairement à une évasion de conteneur à noyau partagé.

## Références

- [Architecture de Kata Containers](https://github.com/kata-containers/kata-containers/blob/main/docs/design/architecture/README.md)
- [Modèle de menace de Kata Containers](https://github.com/kata-containers/kata-containers/blob/main/docs/threat-model/threat-model.md)
