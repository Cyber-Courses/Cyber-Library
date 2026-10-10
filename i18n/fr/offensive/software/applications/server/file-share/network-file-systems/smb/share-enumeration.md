---
title: "Énumération des partages : lister les partages SMB et les permissions"
order: 3
description: "Énumérer les partages exposés par un serveur SMB et l'accès en lecture/écriture dont dispose l'identité courante sur chacun, avec une session nulle, invitée ou valide. La carte d'accès sépare les partages lisibles à piller des partages inscriptibles à empoisonner, et les commentaires et noms de partages révèlent le rôle du serveur et ses données, orientant toutes les actions SMB suivantes."
keywords:
  - smb share enumeration
  - smbclient
  - netexec
  - permissions de partage
  - spider
---

# Énumération des partages

Lister les partages d'un serveur SMB et l'accès de l'identité courante à chacun est la première action et celle qui oriente toutes les autres. Un partage est l'exportation nommée d'un répertoire (ou une ressource spéciale comme `IPC$` pour les tubes nommés), et l'accès est régi à la fois par les permissions au niveau du partage et par les permissions NTFS sous-jacentes, de sorte que l'accès effectif en lecture/écriture pour une identité donnée doit être testé et non supposé. La sortie sépare les partages lisibles, qui deviennent du butin, des partages inscriptibles, qui deviennent des cibles d'empoisonnement et de dépose, et les noms et commentaires des partages révèlent souvent le rôle du serveur (serveur de fichiers, cible de sauvegarde, partage de déploiement).

```bash
# list shares and per-share READ/WRITE for the identity (null, guest, or creds)
nxc smb <target> -u '' -p '' --shares          # anonymous
nxc smb <target> -u user -p 'pass' --shares     # authenticated
nxc smb <target> -u user -H <nthash> --shares   # pass-the-hash
smbclient -L //<target>/ -N                      # list anonymously (no access map)
# connect to a specific share to confirm access and browse
smbclient //<target>/<share> -U 'DOM\user%pass'
#   smb: \> ls ; recurse ON ; prompt OFF ; mget *
```

Lisez la sortie de NetExec colonne par colonne : les indicateurs `READ` et `WRITE` par partage sont la décision, et la liste des partages sépare les partages administratifs par défaut des données métier.

## Parcourir et cartographier le contenu

```bash
# recursively list files across shares to find loot without downloading everything
nxc smb <target> -u user -p 'pass' -M spider_plus          # writes a JSON inventory
nxc smb <target> -u user -p 'pass' -M spider_plus -o DOWNLOAD_FLAG=True READ_ONLY=False
# manual recursive listing of one share
smbclient //<target>/<share> -U 'user%pass' -c 'recurse ON; ls'
```

## Notes d'exploitation

- La carte READ/WRITE est tout l'enjeu de cette étape : `--shares` marque chacun, de sorte qu'un partage inscriptible non par défaut signale une cible d'[empoisonnement](writable-share-poisoning/index.md) et un partage lisible une cible de [pillage](loot-and-sensitive-files/index.md).
- Ignorez les partages par défaut (`C$`, `ADMIN$`, `IPC$`, `SYSVOL`, `NETLOGON`) pour les données, mais notez que l'accès à `C$`/`ADMIN$` implique des droits administratifs (une découverte différente, de plus grande valeur), et que `SYSVOL` est lisible par n'importe quel utilisateur du domaine et héberge la stratégie de groupe (et parfois des identifiants).
- L'accès effectif dépend à la fois des permissions de partage et NTFS ; un partage listé sans indicateur pour une session nulle peut rester lisible avec des identifiants invités ou valides, alors relancez pour chaque identité que vous obtenez.
- `spider_plus` produit un inventaire de fichiers par partage à trier hors ligne avant d'extraire les données, ce qui est plus discret qu'un téléchargement de masse.

## Outils

- [NetExec](https://github.com/Pennyw0rth/NetExec)
- [smbclient (Samba)](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)

## Références

- [NetExec : énumération SMB](https://www.netexec.wiki/smb-protocol/enumeration)
- [MS-SMB2 : TREE_CONNECT](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
