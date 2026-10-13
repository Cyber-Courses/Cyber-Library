---
order: 2
title: "Abus des dossiers partagés et du presse-papiers : s'évader de Parallels via l'intégration Tools"
description: "Parallels Tools fournit des dossiers partagés, un presse-papiers partagé et le glisser-déposer entre l'invité et l'hôte macOS sur les canaux de communication invité-hôte de Parallels. Les gestionnaires côté hôte analysent les requêtes d'intégration fournies par l'invité, et les dossiers partagés exposent des chemins hôte, si bien que l'analyse des requêtes comme la gestion des chemins des dossiers partagés ont produit des évasions guest-to-host et un accès aux fichiers hôte."
keywords:
  - parallels tools
  - shared folders
  - clipboard
  - drag and drop
  - integration
---

# Abus des dossiers partagés et du presse-papiers

Parallels Tools, le paquet d'intégration de l'invité, fournit des dossiers partagés, un presse-papiers partagé et le glisser-déposer entre l'invité et l'hôte macOS. Ceux-ci transitent par les canaux de communication invité-hôte de Parallels, et les gestionnaires côté hôte analysent les requêtes d'intégration de l'invité dans les processus Parallels. Comme avec le GuestRPC/HGFS de VMware et le HGCM de VirtualBox, c'est une surface d'évasion de premier plan : l'analyse des requêtes a donné des évasions par corruption mémoire, et la gestion des chemins des dossiers partagés a permis un accès aux fichiers hôte au-delà du répertoire partagé.

## La surface

```text
Integration surface (Parallels Tools channels):
- shared folders: a host directory exposed into the guest; path resolution runs
  host-side, so traversal/symlink/name-parsing flaws reach host files outside the share
- shared clipboard and drag-and-drop: the host parses guest-supplied formats, sizes,
  and transfer objects; a trusted length/size or a transfer-object lifecycle bug
  (use-after-free) corrupts the host process
- the Tools control channel: integration requests parsed host-side with the same classes
```

```bash
# Parallels Tools and the shared-folder mount (Linux guest)
lsmod 2>/dev/null | grep -i prl
mount 2>/dev/null | grep -i prl_fs       # shared folders via the Parallels filesystem
ls /media/psf 2>/dev/null                 # "psf" = Parallels Shared Folders mount point
```

## Notes d'exploitation

- Deux impacts, comme sur les hyperviseurs comparables : les bugs d'analyse de requêtes donnent l'exécution de code dans le processus hôte Parallels, tandis que la gestion des chemins des dossiers partagés peut donner un accès en lecture/écriture aux fichiers hôte à l'intérieur et au-delà du répertoire partagé.
- La gestion des objets de transfert du glisser-déposer et du presse-papiers est un lieu fréquent (négociation de version, suivi des transferts en cours), l'analogue Parallels des bugs DnD/CP de VMware et HGCM de VirtualBox.
- La fonctionnalité doit être activée pour la surface la plus riche (un dossier partagé configuré, presse-papiers/DnD autorisés), courant sur les VM de bureau ; vérifiez le montage `psf` et les modules Tools.
- Pilotez les canaux depuis un client Tools invité contrôlé ; les évasions de périphériques sont sous [Évasion guest-to-host](guest-to-host-escape.md). Spécifique à une version.

## Références

- [Parallels Desktop : dossiers partagés et Tools](https://www.parallels.com/products/desktop/)
- [Zero Day Initiative : recherche Parallels Tools](https://www.zerodayinitiative.com/blog)
