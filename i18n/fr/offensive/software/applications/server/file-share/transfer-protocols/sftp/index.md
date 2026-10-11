---
title: "SFTP : attaquer le transfert de fichiers basé sur SSH"
description: "SFTP est un sous-système de transfert de fichiers de SSH sur le port 22, il hérite donc de l'authentification de SSH : mots de passe et clés publiques. La surface offensive comprend l'énumération des noms d'utilisateur, la force brute et le spraying de mots de passe, les clés SSH faibles ou volées, et l'évasion du shell restreint ou du chroot auquel les comptes limités à SFTP sont confinés, ce qui transforme l'accès au transfert en exécution de commandes."
keywords:
  - sftp
  - ssh
  - port 22
  - restricted shell
  - chroot
---

# SFTP

SFTP n'est pas du FTP sur SSH mais un sous-système de transfert de fichiers distinct intégré au serveur SSH, atteint sur le port 22 via la connexion SSH. Il hérite donc entièrement du modèle de sécurité de SSH : authentification par mot de passe ou par clé publique, et configuration des comptes et des sous-systèmes du serveur. La surface offensive en découle : énumérer les noms d'utilisateur valides, forcer ou sprayer les mots de passe, abuser des clés SSH faibles ou volées, et, parce que les comptes limités à SFTP sont généralement confinés à un shell restreint `internal-sftp` ou à un chroot, s'évader de ce confinement vers une exécution de commandes complète.

```bash
# SFTP rides SSH; enumerate and connect
nmap -p22 --script ssh2-enum-algos,ssh-auth-methods <target>
sftp user@<target>                            # subsystem access
ssh user@<target>                             # does the account also get a shell?
```

## Sous-thèmes

- **[Énumération des noms d'utilisateur](username-enumeration.md)** : découvrir les comptes valides.
- **[Force brute des identifiants](credential-brute-force.md)** : attaques par mot de passe sur SSH.
- **[Clés SSH faibles et volées](weak-and-stolen-ssh-keys.md)** : abus de l'accès basé sur les clés.
- **[Évasion du shell restreint et du chroot](restricted-shell-and-chroot-escape.md)** : sortir du confinement limité à SFTP.

## Références

- [sftp-server et internal-sftp d'OpenSSH](https://man.openbsd.org/sftp-server.8)
- [Brouillon de RFC : SSH File Transfer Protocol](https://datatracker.ietf.org/doc/html/draft-ietf-secsh-filexfer)
- [HackTricks : SSH (22)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
