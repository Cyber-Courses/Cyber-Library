---
order: 2
title: "Abus des dossiers partagés et du glisser-déposer : s'évader via l'intégration de bureau"
description: "Les fonctionnalités d'intégration de Workstation et Fusion, dossiers partagés (HGFS), glisser-déposer et copier-coller, sont implémentées comme des gestionnaires GuestRPC dans vmware-vmx qui analysent des requêtes complexes fournies par l'invité. Les dossiers partagés exposent aussi des chemins hôtes dans l'invité. La gestion des chemins comme l'analyse RPC ont produit à plusieurs reprises des évasions invité-vers-hôte et des accès aux fichiers de l'hôte."
keywords:
  - hgfs
  - dossiers partagés
  - glisser-déposer
  - copier-coller
  - guestrpc
---

# Abus des dossiers partagés et du glisser-déposer

Les commodités qui rendent les VM de bureau agréables, dossiers partagés, glisser-déposer et copier-coller, sont aussi une surface d'évasion. Les dossiers partagés utilisent le protocole Host-Guest File System (HGFS), et le glisser-déposer et le copier-coller (DnD/CP) utilisent leurs propres protocoles, tous véhiculés via GuestRPC et analysés dans le processus `vmware-vmx`. Deux problèmes en découlent. Les gestionnaires de protocole analysent des requêtes complexes, imbriquées et contrôlées par l'invité dans le processus hôte, une source récurrente d'évasions par corruption mémoire. Et les dossiers partagés exposent des répertoires hôtes dans l'invité, de sorte qu'un path-traversal dans la gestion des chemins HGFS atteint des fichiers hôtes en dehors du répertoire partagé.

## Gestion des chemins HGFS

```bash
# when a shared folder is enabled, it is mounted in the guest via vmhgfs/FUSE
mount | grep -i hgfs; ls /mnt/hgfs/ 2>/dev/null
# the HGFS protocol resolves guest-supplied paths against the shared host directory;
# a traversal in request path handling (../ sequences, symlink tricks, name parsing)
# can escape the shared root and read/write host files outside it
```

## Analyse RPC du DnD/CP et de HGFS

```bash
# drag-and-drop, copy-paste, and HGFS requests travel over GuestRPC (the backdoor)
# the vmx-side handlers parse length-prefixed, nested structures; a length or index
# the handler trusts, or a use-after-free on a transfer object, corrupts vmware-vmx.
# a guest speaks these RPCs directly (see Backdoor and VMCI) without the Tools UI.
```

La négociation de version du DnD et du copier-coller et le cycle de vie des objets de transfert ont été particulièrement productifs, avec des bugs de use-after-free et de hors-limites dans la manière dont les gestionnaires suivent les transferts en cours.

## Notes d'exploitation

- Deux impacts distincts : la gestion des chemins HGFS peut donner la lecture/écriture de fichiers hôtes à l'intérieur et au-delà du dossier partagé, tandis que les bugs d'analyse RPC donnent l'exécution de code complète dans `vmware-vmx`.
- Ces gestionnaires sont accessibles directement via GuestRPC, donc l'attaque ne dépend pas de l'utilisation de l'interface Tools ; un pilote invité émet les RPC forgés, voir [Backdoor et VMCI](../esxi/guest-to-host-escape/backdoor-and-vmci.md).
- La fonctionnalité doit être activée pour la surface la plus riche (un dossier partagé configuré, DnD/CP autorisés), ce qui est courant sur les VM de bureau ; vérifiez les paramètres de la VM ou les montages de l'invité.
- Le même code HGFS/DnD existe sur ESXi, donc ces bugs se recoupent souvent ; cette surface est simplement plus exposée sur les produits de bureau.

## Références

- [open-vm-tools HGFS implementation](https://github.com/vmware/open-vm-tools)
- [Zero Day Initiative: VMware DnD/HGFS research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
