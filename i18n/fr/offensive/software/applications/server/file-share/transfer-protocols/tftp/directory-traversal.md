---
title: "Traversée de répertoires : s'échapper de la racine TFTP"
order: 2
description: "Un serveur TFTP est censé ne servir que son répertoire racine configuré, mais les implémentations qui n'assainissent pas le nom de fichier demandé permettent la traversée de chemin, à l'aide de séquences ../ ou de chemins absolus, pour lire et écrire des fichiers partout où le processus serveur peut atteindre. Cela transforme un service de démarrage d'équipement en lecture arbitraire de fichiers de l'hôte et, là où c'est accessible en écriture, en écriture."
keywords:
  - tftp traversal
  - path traversal
  - ../
  - absolute path
  - arbitrary file read
---

# Traversée de répertoires

Les serveurs TFTP devraient confiner les requêtes à leur répertoire racine, mais de nombreuses implémentations (en particulier sur les équipements embarqués et les démons plus anciens) n'assainissent pas correctement le nom de fichier demandé. Lorsque c'est le cas, un attaquant inclut des séquences `../` ou un chemin absolu dans le nom de fichier RRQ/WRQ pour s'échapper de la racine et lire ou écrire tout fichier auquel le processus serveur peut accéder. Cela fait passer TFTP du service d'un ensemble fixe de fichiers de démarrage à la lecture arbitraire de fichiers, et, si les écritures sont autorisées, à l'écriture arbitraire de fichiers sur l'hôte.

```bash
# attempt traversal reads (syntax varies by server; try both forms)
curl -s tftp://<target>/../../../../etc/passwd -o passwd && cat passwd
tftp <target> -c get ../../../../etc/shadow shadow 2>/dev/null
# absolute-path form, where the server honours it
curl -s tftp://<target>//etc/passwd -o passwd
# traversal write (if writes allowed and sanitisation is absent)
curl -s -T key tftp://<target>/../../../../root/.ssh/authorized_keys
```

## Notes d'exploitation

- Essayez à la fois la traversée `../` et les requêtes par chemin absolu ; différents serveurs gèrent mal l'une ou l'autre, et les démons TFTP embarqués sont des coupables fréquents.
- La lecture arbitraire vise les secrets habituels de l'hôte (`/etc/shadow`, clés SSH, configurations d'applications) accessibles aux privilèges du processus TFTP ; TFTP s'exécute souvent avec des privilèges sur les équipements.
- L'écriture arbitraire, lorsque l'assainissement est absent et les écritures activées, est le résultat le plus puissant : écrivez un `authorized_keys`, une entrée cron ou un script de démarrage que l'hôte exécute.
- Les privilèges du processus bornent la portée ; sur de nombreux équipements embarqués, TFTP s'exécute en tant que root, de sorte que la lecture/écriture par traversée est en pratique sans restriction sur l'équipement.

## Références

- [RFC 1350 (TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [HackTricks : traversée TFTP](https://book.hacktricks.xyz/network-services-pentesting/69-udp-tftp)
