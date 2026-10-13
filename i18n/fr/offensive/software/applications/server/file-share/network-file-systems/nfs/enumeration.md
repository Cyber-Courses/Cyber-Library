---
title: "Énumération : exports NFS, options et montage"
order: 1
description: "L'énumération NFS liste les chemins exportés, les clients autorisés à les monter et, quand c'est lisible, les options d'export qui décident du squashing et de l'accès. Monter un export expose ses fichiers sous l'identité affirmée par le client, et la propriété des fichiers observée après montage révèle quels UID et GID usurper pour un accès plus complet."
keywords:
  - showmount
  - exports
  - mount
  - nfsstat
  - mappage uid
---

# Énumération

Énumérer NFS répond à ce qui est exporté, à qui et sous quelles options, puis le montage révèle la propriété des fichiers qui gouverne les attaques d'usurpation et de squash. La liste des exports et la spécification des clients autorisés viennent du serveur ; les options critiques (`root_squash` vs `no_root_squash`, `all_squash`, `sec=`) ne sont pas toujours visibles à distance, donc monter et inspecter la propriété est la façon d'apprendre le modèle effectif.

```bash
# list exports and permitted clients
showmount -e <target>                          # e.g. /srv/share *  (world) or 10.0.0.0/8
rpcinfo -p <target>                            # confirm mountd/nfs reachable
# mount an export and inspect
mkdir /mnt/nfs && mount -t nfs -o vers=3 <target>:/srv/share /mnt/nfs
ls -lan /mnt/nfs                               # numeric UID/GID ownership of files
mount -t nfs -o vers=4 <target>:/ /mnt/nfs     # NFSv4 presents a single pseudo-root
```

Lisez `ls -lan` de façon numérique : l'UID/GID qui possède chaque fichier est ce que vous devez présenter pour le lire ou l'écrire sous `AUTH_SYS`. Les fichiers possédés par l'UID 0 testent si root est squashé ; les fichiers possédés par d'autres UID nomment les identités à usurper.

## Notes d'exploitation

- `showmount -e` avec un joker ou une spécification de client large signifie que quiconque peut atteindre 2049 peut monter ; une liste de clients restreinte peut être contournable en usurpant l'adresse source sur un réseau à plat.
- Les options d'export sont la vraie sécurité ; comme elles ne sont souvent pas visibles à distance, montez et testez, créez un fichier et voyez sous quel UID il atterrit, et lisez un fichier possédé par root, pour apprendre si `root_squash`/`all_squash` s'appliquent.
- NFSv4 utilise un unique pseudo-système de fichiers exporté et un mappage d'identité basé sur les noms plutôt que le modèle numérique par export de NFSv3 ; montez les deux versions pour voir ce que chacune expose.
- La carte de propriété alimente l'[usurpation d'UID et de GID](uid-and-gid-spoofing.md) et le test [no_root_squash](no-root-squash-abuse.md).

## Outils

- [nfs-common (showmount, mount.nfs)](https://man7.org/linux/man-pages/man8/showmount.8.html)
- [nmap nfs scripts](https://nmap.org/nsedoc/)

## Références

- [exports(5)](https://man7.org/linux/man-pages/man5/exports.5.html)
- [HackTricks: NFS](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
