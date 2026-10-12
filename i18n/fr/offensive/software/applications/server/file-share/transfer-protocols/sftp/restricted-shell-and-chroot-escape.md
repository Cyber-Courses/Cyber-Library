---
title: "Évasion du shell restreint et du chroot : sortir du confinement limité à SFTP"
order: 4
description: "Les comptes limités à SFTP sont confinés par le sous-système internal-sftp d'OpenSSH et généralement par un ChrootDirectory, censés autoriser le transfert de fichiers mais pas l'exécution de commandes. Les évasions proviennent de chemins accessibles en écriture à l'intérieur du chroot sur lesquels le système agit, de fonctionnalités SFTP qui atteignent l'extérieur, d'une propriété de chroot mal configurée, et de l'enchaînement d'une écriture SFTP vers une clé ou une tâche cron que l'hôte exécute avec un vrai shell."
keywords:
  - internal-sftp
  - chrootdirectory
  - restricted shell
  - escape
  - forcecommand
---

# Évasion du shell restreint et du chroot

Les comptes destinés uniquement au transfert de fichiers sont confinés de deux manières dans OpenSSH : `ForceCommand internal-sftp` (ou le sous-système `sftp`) les limite au protocole SFTP sans shell, et `ChrootDirectory` verrouille leur vue du système de fichiers à un sous-arbre. L'objectif d'une évasion est de transformer cet accès limité au transfert en exécution de commandes sur l'hôte. Les voies exploitent ce que le confinement ne couvre pas : les emplacements accessibles en écriture à l'intérieur du chroot sur lesquels un processus extérieur agit, les opérations SFTP qui dépassent la zone prévue, et les mauvaises configurations de chroot.

## Voies

```bash
# 1. the account also has shell access despite SFTP intent? test it:
ssh user@<target> id                      # if a shell returns, there is nothing to escape
# 2. write a key/cron/script the HOST executes with a real shell, via SFTP write:
#    - if your chroot home maps to a real user home, upload .ssh/authorized_keys then SSH
sftp user@<target> <<'E'
put authorized_keys .ssh/authorized_keys
E
ssh -i attacker_key user@<target>         # now a full session if the key is honoured
# 3. chroot misconfiguration: ChrootDirectory must be root-owned and not writable by
#    the user; if the chroot root (or a parent) is user-writable, it can be abused,
#    and writable system paths inside the chroot (cron.d, scripts run by root) execute
# 4. SFTP symlink/hardlink tricks to reference files outside the intended subtree
sftp> symlink / escape        # then browse "escape" if the server resolves it host-side
```

## Notes d'exploitation

- Confirmez d'abord que le confinement est réel : beaucoup de comptes « limités à SFTP » accordent en réalité encore un shell (`ssh user@host id`), auquel cas il n'y a aucune évasion à réaliser.
- L'évasion la plus fiable est indirecte : utilisez l'écriture SFTP pour déposer quelque chose que l'hôte exécute avec un vrai shell, un `authorized_keys` (si le home du chroot est le home réel), un fichier dans un chemin `cron.d`/script accessible en écriture, ou un fichier web si le chroot chevauche une racine web.
- Un `ChrootDirectory` correctement configuré appartient à root et n'est pas accessible en écriture par l'utilisateur sur tout le chemin ; les violations de cela (chroot ou parent accessible en écriture par l'utilisateur) constituent une faiblesse directe.
- Lorsque le compte correspond à un utilisateur système dont vous pouvez écrire dans le home ou les tâches planifiées, l'accès de transfert se convertit en exécution en tant que cet utilisateur ; enchaînez avec la technique des [clés](weak-and-stolen-ssh-keys.md).

## Références

- [OpenSSH : ChrootDirectory et internal-sftp](https://man.openbsd.org/sshd_config#ChrootDirectory)
- [HackTricks : évasion de shell restreint SSH](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
