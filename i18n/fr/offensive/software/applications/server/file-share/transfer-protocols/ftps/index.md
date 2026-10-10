---
title: "FTPS : attaquer FTP sur TLS"
description: "FTPS ajoute TLS à FTP, soit implicitement sur le port 990, soit explicitement via AUTH TLS sur le port 21. Il protège les identifiants et les données que le FTP simple expose, mais uniquement lorsque le TLS est correctement configuré et vérifié. La surface offensive comprend un TLS faible ou non vérifié permettant l'interception, et les faiblesses FTP sous-jacentes (accès anonyme, brute force) que TLS ne supprime pas."
keywords:
  - ftps
  - auth tls
  - port 990
  - tls
  - ftp over tls
---

# FTPS

FTPS est FTP encapsulé dans TLS. Il existe sous deux formes : le FTPS implicite, qui négocie TLS immédiatement à la connexion sur le port 990, et le FTPS explicite, qui démarre en clair sur le port 21 et bascule avec la commande `AUTH TLS`. Bien réalisé, il corrige le plus gros défaut de FTP, les identifiants et les données en clair, mais la protection dépend entièrement de la configuration du TLS et de sa vérification par le client. La surface offensive est donc double : un TLS faible ou non vérifié qui permet encore l'interception, et les faiblesses au niveau FTP (accès anonyme, brute force, rebond) que le chiffrement ne traite pas.

```bash
# detect FTPS and inspect its TLS
nmap -p21,990 --script ftp-syst,ssl-enum-ciphers <target>
openssl s_client -connect <target>:990           # implicit FTPS
# explicit: connect plain to 21, then AUTH TLS, then STARTTLS handshake
curl -v --ssl-reqd ftp://<target>/ --user user:pass
```

## Sous-thèmes

- **[TLS faible et validation de certificat](weak-tls-and-certificate-validation.md)** : interception par une mauvaise configuration TLS.

## Références

- [RFC 4217 (Securing FTP with TLS)](https://datatracker.ietf.org/doc/html/rfc4217)
- [HackTricks : FTP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
