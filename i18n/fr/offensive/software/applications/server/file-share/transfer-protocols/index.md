---
title: "Protocoles de transfert : attaquer les services de transfert de fichiers"
description: "Les services de transfert de fichiers déplacent des fichiers via des protocoles dédiés : FTP et sa variante TLS FTPS, SFTP sur SSH, le trivial TFTP, et rsync. Chacun a ses propres faiblesses, accès anonyme et par défaut, brute force d'identifiants, traversée de répertoires, sécurité de transport faible, et évasions de shell restreint, qui transforment un point de terminaison de transfert en accès aux données ou en point d'appui."
keywords:
  - ftp
  - sftp
  - tftp
  - rsync
  - file transfer
---

# Protocoles de transfert

Au-delà des systèmes de fichiers réseau montables, les fichiers transitent via des services de transfert dédiés, et chacun constitue sa propre surface d'attaque. FTP est ancien et en clair, avec un accès anonyme et des particularités de rebond ; FTPS l'encapsule dans un TLS souvent mal configuré ; SFTP s'appuie sur SSH et hérite de ses faiblesses d'identifiants et de clés ainsi que des évasions de shell restreint ; TFTP est un petit protocole UDP sans authentification sujet à la traversée ; et les démons rsync exposent fréquemment des modules de manière anonyme. Les manœuvres récurrentes sont l'accès anonyme ou par défaut, les attaques sur les identifiants, la traversée de répertoires, la sécurité de transport faible, et l'évasion d'un shell restreint réservé au transfert vers un shell réel.

```bash
# identify transfer services on a host
nmap -p21,22,69,873,990 -sV <target>        # FTP, SSH/SFTP, TFTP, rsync, FTPS
nmap -p21 --script ftp-anon,ftp-syst <target>
```

## Sous-thèmes

- **[FTP](ftp/index.md)** : accès anonyme, brute force d'identifiants et particularité du rebond.
- **[FTPS](ftps/index.md)** : FTP sur TLS et ses faiblesses de certificat et de configuration.
- **[SFTP](sftp/index.md)** : transfert basé sur SSH, clés et évasion de shell restreint.
- **[TFTP](tftp/index.md)** : le trivial protocole sans authentification et la traversée.
- **[rsync](rsync/index.md)** : modules du démon rsync et accès anonyme.

## Références

- [RFC 959 (FTP)](https://datatracker.ietf.org/doc/html/rfc959)
- [RFC 913 / RFC 1350 (TFTP)](https://datatracker.ietf.org/doc/html/rfc1350)
- [rsync documentation](https://rsync.samba.org/documentation.html)
