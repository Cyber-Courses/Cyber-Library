---
title: "Énumération : identification d'un service Telnet"
order: 3
description: "L'énumération Telnet lit la bannière du service et l'invite de connexion, qui sont souvent bavardes, divulguant le système d'exploitation, le modèle de l'équipement et la version du logiciel, et parfois un avertissement avant connexion ou un détail de configuration. Cette identification associe la cible aux identifiants par défaut et aux failles de corruption mémoire de telnetd qu'elle peut comporter."
keywords:
  - énumération telnet
  - bannière
  - détection d'os
  - invite de connexion
  - fingerprint
---

# Énumération

Telnet est bavard : à la connexion il présente généralement une bannière et une invite de connexion, et les deux divulguent fréquemment le système d'exploitation, le fournisseur et le modèle de l'équipement, et la version du logiciel, certains systèmes ajoutant des messages avant connexion qui révèlent la configuration ou la finalité. Cette identification conditionne la suite : l'équipement et le système d'exploitation correspondent directement aux identifiants par défaut connus et aux vulnérabilités de corruption mémoire de l'implémentation `telnetd`, et le format de l'invite de connexion lui-même peut révéler des spécificités du système d'exploitation utiles pour l'identification.

```bash
nc <target> 23                                 # banner + login prompt
nmap -p23 -sV <target>                         # service/version
nmap -p23 --script telnet-ntlm-info <target>   # NTLM info on Windows telnet
```

## Sous-thèmes

- **[Capture de bannière](banner-grabbing.md)** : la bannière du service et la version.
- **[Détection d'OS](os-detection.md)** : identifier le système d'exploitation à partir de la bannière et de l'invite.

## Références

- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
