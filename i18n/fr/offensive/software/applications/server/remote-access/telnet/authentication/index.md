---
order: 1
title: "Authentification : attaquer le login Telnet"
description: "Telnet s'authentifie en clair avec un nom d'utilisateur et un mot de passe, et il est attaqué via des identifiants par défaut constructeur (omniprésents sur les équipements), des serveurs configurés sans authentification ou donnant un shell direct, et la force brute contre le login en clair. Un login valide donne un terminal, souvent sur de l'équipement réseau ou un appareil embarqué avec une portée administrative."
keywords:
  - authentification telnet
  - identifiants par défaut
  - aucune authentification
  - force brute
  - clair
---

# Authentification

L'authentification de Telnet est un échange en clair de nom d'utilisateur et de mot de passe, et elle cède de trois façons. Les identifiants par défaut sont endémiques sur les équipements qui exécutent encore Telnet, routeurs, switchs, caméras, imprimantes, IoT, donc la paire constructeur documentée fonctionne souvent. Certains serveurs ne requièrent aucune authentification, donnant directement un shell ou un menu. Et le login en clair peut être forcé par force brute, borné principalement par la limitation de débit (souvent absente) de l'équipement. Un login réussi donne un terminal, fréquemment sur de l'équipement où ce terminal est administratif (un routeur capable de passer en enable, un shell root BusyBox).

```bash
nc <target> 23        # try default/no-auth first
hydra -L users.txt -P passwords.txt telnet://<target> -t 4
nxc telnet <target> -u users.txt -p passwords.txt 2>/dev/null
```

## Sous-thèmes

- **[Identifiants par défaut](default-credentials.md)** : logins Telnet constructeur et d'usine.
- **[Aucune authentification](no-authentication.md)** : serveurs qui donnent un shell sans mot de passe.
- **[Force brute de mot de passe](password-brute-force.md)** : attaques en ligne sur le login en clair.

## Références

- [HackTricks: Telnet authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
