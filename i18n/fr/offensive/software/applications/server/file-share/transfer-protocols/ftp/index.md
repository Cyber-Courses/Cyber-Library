---
title: "FTP : attaquer le File Transfer Protocol"
description: "FTP sert des fichiers sur le port TCP 21 en clair, avec un canal de contrôle et des connexions de données séparées. La surface offensive comprend l'accès anonyme que de nombreux serveurs autorisent, le brute force d'identifiants contre sa connexion en clair, la fonction FTP bounce qui relaie des scans de ports à travers le serveur, et la capture d'identifiants depuis son trafic non chiffré."
keywords:
  - ftp
  - port 21
  - anonymous
  - ftp bounce
  - cleartext
---

# FTP

FTP (File Transfer Protocol) sert des fichiers sur le port TCP 21 à l'aide d'un canal de contrôle en clair et de connexions de données séparées (actives ou passives). Son âge se reflète dans sa sécurité : les identifiants et les données transitent sans chiffrement, l'accès anonyme est une fonction intégrée fréquemment laissée activée, et la commande `PORT` rend possible la technique de rebond qui transforme le serveur en proxy de scan. La surface est donc l'accès anonyme, le brute force contre la connexion en clair, la particularité du rebond et la capture passive d'identifiants et de fichiers depuis le flux non chiffré.

```bash
nmap -p21 --script ftp-anon,ftp-syst,ftp-bounce <target>
ftp <target>                                  # interactive; try anonymous first
```

## Sous-thèmes

- **[Accès anonyme](anonymous-access.md)** : la connexion anonyme intégrée.
- **[Brute force d'identifiants](credential-brute-force.md)** : attaquer la connexion en clair.
- **[FTP bounce scan](ftp-bounce-scan.md)** : relayer des scans de ports à travers le serveur.

## Références

- [RFC 959 (FTP)](https://datatracker.ietf.org/doc/html/rfc959)
- [HackTricks : FTP (21)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
