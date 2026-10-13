---
title: "Sauvegardes et archives : exploiter les systèmes stockés pour une compromission hors ligne"
order: 2
description: "Les partages utilisés comme cibles de sauvegarde contiennent des systèmes entiers : images disque de VM, exports de bases de données, sauvegardes d'état système et de registre, exports de boîtes aux lettres et archives. Un accès en lecture à ces données permet à un attaquant de reconstruire des systèmes hors ligne, en extrayant des empreintes de mots de passe depuis les ruches de registre, des secrets depuis les exports de bases de données et des fichiers depuis les images, sans toucher aux systèmes vivants dont elles proviennent."
keywords:
  - backups
  - vmdk
  - registry hive
  - database dump
  - archive
---

# Sauvegardes et archives

Les partages de sauvegarde comptent parmi les butins les plus riches, car une sauvegarde est une copie complète d'un système, lisible hors ligne sans aucune des défenses du système vivant. Une cible de sauvegarde lisible fournit des images disque de VM, des sauvegardes d'état système et de registre, des exports de bases de données, des exports de boîtes aux lettres et des archives générales, chacune reconstructible en identifiants et en données. La méthode consiste à identifier les artefacts de sauvegarde, à les récupérer et à les extraire hors ligne.

## Identifier et récupérer

```bash
# backup artefacts commonly found on backup shares
find ./loot -iregex '.*\.\(bak\|vhd\|vhdx\|vmdk\|vbk\|ova\|tar\|zip\|7z\|gz\|bkf\|wbcat\)$'
#   plus: *.ldf/*.mdf (SQL), *.bacpac/*.dacpac, *.pst/*.ost (mailboxes),
#         ntds.dit, SYSTEM/SECURITY/SAM hives, System State backups
```

## Extraire les identifiants et les données hors ligne

```bash
# Windows registry hives from a backup -> local account hashes and LSA secrets
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL
# a domain controller backup containing ntds.dit -> every domain hash
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
# VM disk images -> mount and read like any disk theft
guestmount -a system.vmdk -i --ro /mnt && ls /mnt/Windows/System32/config
# database dumps -> restore or grep for secrets and data
strings backup.bak | grep -iE 'password|secret|connectionstring' | head
```

Une sauvegarde d'un contrôleur de domaine incluant `ntds.dit` plus la ruche `SYSTEM` est le jackpot : `secretsdump` extrait hors ligne l'empreinte de chaque compte du domaine, ce qui constitue une compromission à l'échelle du domaine à partir d'une seule archive lisible.

## Notes d'exploitation

- La combinaison `ntds.dit` + `SYSTEM` issue de n'importe quelle sauvegarde de DC fournit hors ligne toutes les empreintes du domaine ; donnez la priorité aux partages de sauvegarde et aux sauvegardes d'état système pour cela.
- Les jeux de ruches locales (`SAM` + `SYSTEM` + `SECURITY`) donnent les empreintes d'administrateur local et les secrets LSA (qui incluent souvent les mots de passe des comptes de service sous une forme équivalente au texte clair).
- Les images de VM et les fichiers `.vbk`/`.vmdk` se lisent avec les mêmes techniques de montage hors ligne que le vol de disque d'hyperviseur ; privilégiez les magasins d'identifiants à l'intérieur.
- Tout ici se fait hors ligne et ne touche aucun système vivant, c'est donc peu bruyant ; la seule contrainte est l'accès en lecture au partage de sauvegarde.

## Outils

- [Impacket secretsdump](https://github.com/fortra/impacket)
- [libguestfs / guestmount](https://libguestfs.org/)

## Références

- [MITRE ATT&CK : données issues des sauvegardes](https://attack.mitre.org/techniques/T1530/)
- [Utilisation d'Impacket secretsdump](https://github.com/fortra/impacket)
