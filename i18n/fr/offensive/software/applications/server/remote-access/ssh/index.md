---
order: 3
title: "SSH : attaquer le service Secure Shell"
description: "SSH sur le TCP 22 fournit une administration à distance chiffrée, il résiste donc à l'attaque passive mais expose une riche surface active : énumération de service et d'utilisateurs, attaques d'authentification par mot de passe et par clé, négociation et downgrade de crypto faible, abus de confiance de clé d'hôte et de certificat pour le machine-in-the-middle, et tunneling qui transforme un seul point d'appui SSH en pivot réseau."
keywords:
  - ssh
  - openssh
  - port 22
  - authentification par clé
  - tunneling
---

# SSH

SSH (Secure Shell) est le protocole standard d'administration à distance chiffrée, à l'écoute sur le TCP 22, et parce que la session est chiffrée et protégée en intégrité, la surface d'attaque est active plutôt que passive. Cinq aspects comptent. L'énumération identifie la version, les algorithmes pris en charge et les utilisateurs valides. L'authentification est attaquée via les mots de passe, les identifiants par défaut et les clés qui accordent réellement l'accès. La cryptographie faible permet à un attaquant bien positionné de négocier ou de rétrograder vers des chiffrements, MAC ou échanges de clés cassables. La confiance de clé d'hôte et de certificat, bâtie sur le trust-on-first-use, est abusée pour le machine-in-the-middle. Et les fonctionnalités de tunneling de SSH transforment un seul hôte atteignable en pivot sur le réseau.

```bash
# fingerprint the service
nmap -p22 -sV --script ssh2-enum-algos,ssh-hostkey,ssh-auth-methods <target>
nc <target> 22                                 # banner (SSH-2.0-OpenSSH_x.y)
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : bannière, algorithmes et utilisateurs valides.
- **[Authentification](authentication/index.md)** : mots de passe, valeurs par défaut, clés et clés faibles.
- **[Cryptographie faible](weak-cryptography/index.md)** : faiblesses de chiffrement, de MAC et d'échange de clés.
- **[MITM et confiance](mitm-and-trust/index.md)** : abus de confiance de clé d'hôte et de certificat.
- **[Tunneling](tunneling/index.md)** : redirection de port, SOCKS, hôtes de rebond et détournement d'agent.

## Références

- [OpenSSH](https://www.openssh.com/manual.html)
- [RFC 4251-4254 (SSH architecture and protocols)](https://datatracker.ietf.org/doc/html/rfc4251)
- [HackTricks: SSH (22)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
