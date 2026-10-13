---
title: "TFTP : attaquer le Trivial File Transfer Protocol"
description: "TFTP est un protocole de transfert de fichiers minimal en UDP sur le port 69, sans aucune authentification. N'importe quel client peut lire et, là où le serveur l'autorise, écrire des fichiers. La surface offensive comprend la lecture des fichiers sensibles que le serveur expose (configurations, firmwares, fichiers de démarrage PXE), l'écriture pour influencer un équipement qui les consomme, et la traversée de répertoires pour s'échapper de la racine TFTP."
keywords:
  - tftp
  - port 69
  - udp
  - no authentication
  - pxe
---

# TFTP

TFTP (Trivial File Transfer Protocol) est un protocole de transfert de fichiers délibérément minimal sur le port UDP 69. Il n'a aucune authentification, aucun listage de répertoire, et seulement deux véritables opérations : lire un fichier (RRQ) et écrire un fichier (WRQ). Il est utilisé là où la simplicité compte, démarrage réseau PXE, firmware et configuration de routeurs et de téléphones, sauvegarde de configurations d'équipements, de sorte que les fichiers qu'il sert sont souvent sensibles, et un serveur TFTP accessible en écriture permet à un attaquant d'influencer l'équipement qui consomme ces fichiers. Comme il n'y a pas d'authentification, atteindre le port 69 est l'unique condition préalable.

```bash
# connect (no listing exists; you request known/guessed filenames)
tftp <target>
#   tftp> get <filename>      # read
#   tftp> put <localfile> <remote>   # write (if allowed)
# scripted
curl -s tftp://<target>/<filename> -o out
nmap -sU -p69 --script tftp-enum <target>     # probe for common filenames
```

## Sous-thèmes

- **[Lecture et écriture de fichiers](file-read-and-write.md)** : lire des fichiers sensibles et écrire sur le serveur.
- **[Traversée de répertoires](directory-traversal.md)** : s'échapper de la racine TFTP par traversée de chemin.

## Références

- [RFC 1350 (TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [HackTricks : TFTP (69)](https://book.hacktricks.xyz/network-services-pentesting/69-udp-tftp)
