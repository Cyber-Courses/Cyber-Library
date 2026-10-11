---
title: "File Share : attaquer les services de partage et de transfert de fichiers"
order: 4
description: "Attaquer les services de partage et de transfert de fichiers : systèmes de fichiers réseau (SMB, NFS, AFP) atteints par montage ou par des clients de protocole, protocoles de transfert (FTP, FTPS, SFTP, TFTP, rsync), et accès web aux fichiers (serveurs de fichiers HTTP, WebDAV, appliances de transfert de fichiers managé). Couvre l'accès anonyme et faible, l'abus de partages accessibles en écriture et le butin."
keywords:
  - partage de fichiers
  - SMB
  - NFS
  - FTP
  - transfert de fichiers managé
---

# File Share

Les partages de fichiers sont l'endroit où un environnement garde ses données, et ils sont régulièrement exposés avec une authentification faible ou inexistante, des permissions lisibles ou inscriptibles par tout le monde, et des protocoles hérités. La valeur offensive va dans deux directions : lire ce que le partage contient (identifiants, sauvegardes, documents) et y écrire pour obtenir une exécution ou contraindre d'autres utilisateurs. Le domaine se divise selon la manière dont les fichiers sont atteints.

## Trouver les services de fichiers

Un seul balayage identifie les services de fichiers sur un hôte et pointe vers le bon sous-sujet :

```bash
nmap -sV -p21,22,69,111,139,445,548,873,990,2049,80,443 <target>
#  445/139 SMB   2049/111 NFS   548 AFP   21 FTP   990 FTPS   22 SFTP(SSH)
#  69 TFTP(UDP)  873 rsync   80/443 HTTP file server / WebDAV / MFT portal
# quick access probes
nxc smb <target> -u '' -p '' --shares          # SMB null-session shares
showmount -e <target>                           # NFS exports
rsync <target>::                                # rsync modules
curl -s -X OPTIONS http://<target>/ -i | grep -i dav   # WebDAV
```

Reliez les ports ouverts au sous-sujet : 445/2049/548 vers les [systèmes de fichiers réseau](network-file-systems/index.md), 21/22/69/873 vers les [protocoles de transfert](transfer-protocols/index.md), et 80/443 servant des fichiers vers l'[accès web aux fichiers](web-file-access/index.md). Ensuite, pour chacun, les questions récurrentes sont les mêmes : l'accès est-il anonyme ou faible, est-il lisible (butin) ou inscriptible (exécution et coercition), et l'implémentation du serveur présente-t-elle une faille connue.

## Sous-sujets

- **[Systèmes de fichiers réseau](network-file-systems/index.md)** : SMB, NFS et AFP, atteints par montage ou par des clients de protocole.
- **[Protocoles de transfert](transfer-protocols/index.md)** : FTP, FTPS, SFTP, TFTP et rsync.
- **[Accès web aux fichiers](web-file-access/index.md)** : serveurs de fichiers HTTP, WebDAV et appliances de transfert de fichiers managé.

## Références

- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
