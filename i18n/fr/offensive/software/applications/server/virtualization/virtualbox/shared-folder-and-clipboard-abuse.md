---
order: 2
title: "Abus des dossiers partagés et du presse-papiers : s'évader de VirtualBox via les Guest Additions"
description: "Les Guest Additions de VirtualBox fournissent des dossiers partagés, un presse-papiers partagé et le glisser-déposer via le Host-Guest Communication Manager (HGCM). Les services HGCM côté hôte analysent les requêtes fournies par l'invité dans le processus VM, et les dossiers partagés exposent des chemins hôtes, de sorte que l'analyse HGCM comme la gestion des chemins des dossiers partagés ont produit des évasions guest-to-host et des accès aux fichiers de l'hôte."
keywords:
  - guest additions
  - hgcm
  - dossiers partagés
  - presse-papiers
  - vboxsf
---

# Abus des dossiers partagés et du presse-papiers

Les Guest Additions de VirtualBox ajoutent l'intégration avec l'hôte, dossiers partagés (`vboxsf`), presse-papiers partagé et glisser-déposer, véhiculés via le Host-Guest Communication Manager (HGCM), un canal requête/réponse entre l'invité et le processus VM hôte. Les services HGCM (dossiers partagés, presse-papiers, glisser-déposer, propriétés invité, et plus) analysent les requêtes fournies par l'invité dans le processus hôte, et les dossiers partagés exposent en outre des répertoires hôtes dans l'invité. Les deux sont des surfaces d'évasion : l'analyse des requêtes HGCM a produit des évasions par corruption mémoire, et la gestion des chemins des dossiers partagés a permis un accès aux fichiers de l'hôte au-delà du répertoire partagé.

## La surface HGCM

```c
// a guest issues HGCM calls to a named service with typed parameters (pointers to
// guest buffers with lengths, 32/64-bit values). The host service parses them.
//  - a parameter length/count the service trusts for a copy -> OOB in the VM process
//  - a service-specific request (shared-folder op, clipboard format) with a crafted
//    size or index the handler does not bound
// services: VBoxSharedFolders, VBoxSharedClipboard, VBoxDragAndDrop, VBoxGuestProps
```

```bash
# Guest Additions and the shared-folder mount
lsmod | grep -i vboxguest; mount | grep -i vboxsf
VBoxControl guestproperty enumerate 2>/dev/null   # the guest-properties HGCM service
```

## Gestion des chemins des dossiers partagés

```bash
# a shared folder maps a host directory; its path resolution runs host-side
ls /media/sf_* /mnt/*sf* 2>/dev/null
# path-handling flaws (traversal, symlink, name parsing) in the shared-folder service
# can reach host files outside the shared root, or mishandle a crafted request
```

## Notes d'exploitation

- Deux impacts : les bugs d'analyse des requêtes HGCM donnent l'exécution de code dans le processus VM, tandis que la gestion des chemins des dossiers partagés peut donner la lecture/écriture de fichiers hôtes à l'intérieur et au-delà du répertoire partagé.
- Les services HGCM sont accessibles via l'interface des Guest Additions depuis un invité contrôlé ; un attaquant émet directement des appels HGCM forgés plutôt que d'utiliser l'interface d'intégration normale.
- La fonctionnalité doit être activée pour la surface la plus riche (un dossier partagé configuré, presse-papiers/DnD autorisés), fréquent sur les VM de bureau ; vérifiez les montages et les propriétés invité.
- Il s'agit d'une surface Pwn2Own incontournable de VirtualBox ; les évasions par périphériques sont sous [Évasion guest-to-host](guest-to-host-escape/index.md).

## Références

- [VirtualBox manual: shared folders and Guest Additions](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox HGCM research](https://www.zerodayinitiative.com/blog)
