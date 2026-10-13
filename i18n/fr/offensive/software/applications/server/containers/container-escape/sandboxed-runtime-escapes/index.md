---
title: "Évasions de runtimes en bac à sable : sortir de l'isolation gVisor et Kata"
order: 5
description: "Les runtimes en bac à sable ajoutent une couche entre le conteneur et le noyau de l'hôte : gVisor interpose un noyau en espace utilisateur, et Kata Containers exécute chaque conteneur dans une machine virtuelle légère. S'en évader revient à vaincre cette couche supplémentaire, via un bug dans l'émulation d'appels système et le proxy de fichiers du noyau en espace utilisateur, ou une évasion de VM et l'agent Kata, avant la compromission habituelle de l'hôte."
keywords:
  - gvisor
  - kata containers
  - évasion de bac à sable
  - runtime en bac à sable
  - évasion de conteneur
---

# Évasions de runtimes en bac à sable

Les conteneurs standards partagent directement le noyau de l'hôte, de sorte qu'un bug du noyau est un bug de l'hôte. Les runtimes en bac à sable insèrent une barrière. gVisor exécute un noyau en espace utilisateur (le Sentry) qui intercepte les appels système du conteneur pour qu'ils n'atteignent jamais directement le noyau de l'hôte, avec un proxy de fichiers séparé (le Gofer) qui médie l'accès au système de fichiers. Kata Containers exécute chaque conteneur à l'intérieur d'une machine virtuelle légère dotée de son propre noyau invité, de sorte que le conteneur est isolé de l'hôte par l'hyperviseur. S'évader de l'un ou de l'autre nécessite d'abord de vaincre la couche supplémentaire, et ce n'est qu'ensuite que les techniques familières de l'hôte s'appliquent.

Détectez le runtime utilisé :

```bash
dmesg 2>/dev/null | grep -i gvisor
cat /proc/version 2>/dev/null                 # gVisor reports a distinctive version string
mount | grep -i kata; ls /dev | grep -i kata  # Kata guest artefacts
uname -a                                       # a minimal guest kernel suggests a VM-based sandbox
```

## Sous-thèmes

- **[gVisor](gvisor.md)** : s'évader du noyau en espace utilisateur et de son proxy de fichiers.
- **[Kata Containers](kata-containers.md)** : s'évader de la VM invitée et atteindre l'hôte via l'agent.

## Références

- [Modèle de sécurité de gVisor](https://gvisor.dev/docs/architecture_guide/security/)
- [Architecture de Kata Containers](https://github.com/kata-containers/kata-containers/blob/main/docs/design/architecture/README.md)
