---
title: "Détection d'OS : identifier le système derrière Telnet"
description: "Au-delà de la bannière, le format de l'invite de login Telnet, la négociation des options et les messages post-login prennent l'empreinte du système d'exploitation : une séquence login:/Password: Unix, une invite Windows, une interface Cisco ou BusyBox, ou un menu constructeur. Connaître l'OS oriente les choix d'identifiants, d'exploitation et de post-accès."
keywords:
  - détection d'os
  - invite de login
  - négociation d'options
  - empreinte
  - telnet
---

# Détection d'OS

Telnet divulgue le système d'exploitation par plus que la seule bannière. La séquence de login diffère selon l'OS (`login:`/`Password:` sous Unix, invites Windows distinctes, un `Username:`/`Password:` ou une invite enable Cisco, un shell BusyBox sur du Linux embarqué, ou un menu de configuration constructeur), la négociation des options Telnet (l'échange de commandes IAC à la connexion) varie selon l'implémentation, et les messages post-login (motd, type de shell) confirment la plateforme. Fixer l'OS et le type d'appareil oriente les identifiants à essayer, les failles `telnetd` applicables, et les actions de post-accès qui ont du sens.

```bash
nc <target> 23                                 # observe prompt style and any menu
nmap -p23 -O --script telnet-encryption <target>   # OS hints + option negotiation
# the IAC option negotiation bytes at connect differ per implementation
# Unix: "login:" then "Password:"; Cisco: "Username:"/enable; BusyBox: direct shell/menu
```

## Notes d'exploitation

- Le style de l'invite est un indice rapide d'OS/d'appareil : un simple `login:` suggère Unix/Linux, un menu constructeur suggère une appliance, une invite Cisco suggère de l'équipement réseau avec son propre modèle d'identifiants et d'enable.
- Le comportement de négociation des options et la version prennent l'empreinte de l'implémentation `telnetd`, affinant le ciblage de la [corruption mémoire](../memory-corruption/index.md).
- L'OS/appareil identifié sélectionne l'ensemble d'[identifiants par défaut](../authentication/default-credentials.md) et façonne les attentes de post-accès (une invite enable de routeur, un shell Unix, un menu restreint dont s'échapper).
- Combinez avec la [bannière](banner-grabbing.md) pour une empreinte complète avant de vous authentifier.

## Références

- [RFC 854: option negotiation (IAC)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
