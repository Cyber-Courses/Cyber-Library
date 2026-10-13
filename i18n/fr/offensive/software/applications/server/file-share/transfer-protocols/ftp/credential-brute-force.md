---
title: "Brute force d'identifiants : attaquer la connexion FTP en clair"
description: "L'authentification FTP est un nom d'utilisateur et un mot de passe en clair sans limitation de débit intégrée dans de nombreux serveurs, ce qui en fait une cible directe de brute force et de password spray. Des identifiants valides donnent l'accès aux fichiers de cet utilisateur et, lorsque l'utilisateur FTP correspond à un compte système, souvent un point d'appui réutilisable sur SSH ou d'autres services."
keywords:
  - ftp brute force
  - password spray
  - hydra
  - cleartext
  - credential reuse
---

# Brute force d'identifiants

Les connexions FTP sont des paires utilisateur/mot de passe en clair, et de nombreux serveurs FTP n'appliquent aucun verrouillage ni limitation de débit, ce qui rend le brute force en ligne et le password spraying praticables. Des identifiants valides donnent l'accès aux fichiers de ce compte ; surtout, les utilisateurs FTP correspondent fréquemment à de vrais comptes système, si bien qu'un identifiant FTP fonctionnel est souvent réutilisable pour SSH, SMB ou la connexion au système d'exploitation, transformant l'accès aux fichiers en un point d'appui système.

```bash
# spray a known user list with common passwords (quieter), or brute one account
hydra -L users.txt -p 'Winter2025!' ftp://<target>        # spray one password
hydra -l admin -P rockyou.txt ftp://<target> -t 4 -f      # brute one user, stop on hit
# netexec supports FTP too
nxc ftp <target> -u users.txt -p passwords.txt
```

## Notes d'exploitation

- Préférez pulvériser un mot de passe courant sur de nombreux utilisateurs plutôt que marteler un seul compte, à la fois pour trouver les comptes faibles et pour éviter tout verrouillage existant ; dérivez la liste d'utilisateurs d'autres énumérations (cyclage de RID SMB, OSINT).
- Les comptes FTP sont couramment des comptes système, testez donc immédiatement tout identifiant fonctionnel contre SSH et d'autres services ; la réutilisation d'identifiants est le gain principal.
- La connexion en clair signifie aussi qu'une position d'écoute capture les identifiants sans aucun brute force ; sur un segment partagé, la capture bat la devinette.
- L'accès aux fichiers d'un compte trouvé peut lui-même être l'objectif (un FTP de sauvegarde ou de données), indépendamment de la réutilisation système.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec](https://github.com/Pennyw0rth/NetExec)

## Références

- [HackTricks: FTP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
- [RFC 959: authentication](https://datatracker.ietf.org/doc/html/rfc959)
