---
order: 1
title: "Interception de trafic : capture et rejeu des r-commands en clair"
description: "Les r-commands ne chiffrent rien : un attaquant bien positionné capture les mots de passe envoyés par rexec/rlogin, chaque commande et sa sortie, ainsi que des sessions entières. Les flux en clair, authentifiés uniquement à l'établissement, peuvent être détournés pour injecter des commandes et rejoués, ce qui livre identifiants, données et exécution sans avoir à vaincre l'authentification."
keywords:
  - interception de trafic
  - texte en clair
  - sniffing de mot de passe
  - capture de commande
  - rejeu de session
---

# Interception de trafic

Aucune des r-commands ne chiffre son trafic : un attaquant sur le chemin lit tout ce qu'elles transportent, les mots de passe en clair envoyés par rexec et par rlogin en repli sur mot de passe, chaque commande exécutée et sa sortie complète, et des sessions interactives entières. Au-delà de la capture passive, les flux ne sont authentifiés qu'à l'établissement : ils peuvent donc être détournés pour injecter des commandes au nom de l'utilisateur authentifié, et les sessions capturées peuvent être rejouées. L'interception est souvent l'attaque la plus simple contre les déploiements hérités de r-commands, car elle ne nécessite ni devinette d'identifiant ni manipulation de confiance, seulement une position sur le réseau.

```bash
# capture across the r-command ports
tcpdump -i eth0 -A 'port 512 or port 513 or port 514' -w rcmds.pcap
tshark -r rcmds.pcap -q -z follow,tcp,ascii,0
```

## Sous-thèmes

- **[Sniffing de mot de passe](password-sniffing.md)** : capture des identifiants depuis l'échange en clair.
- **[Interception de commandes](command-interception.md)** : lecture des commandes et de la sortie en transit.
- **[Rejeu de session](session-replay.md)** : rejeu des sessions r-command capturées.

## Références

- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin, cleartext)](https://datatracker.ietf.org/doc/html/rfc1282)
