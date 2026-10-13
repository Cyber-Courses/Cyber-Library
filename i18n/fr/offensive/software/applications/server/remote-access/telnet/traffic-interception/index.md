---
order: 4
title: "Interception de trafic : capturer et détourner du Telnet en clair"
description: "Telnet n'est pas chiffré, de sorte qu'un attaquant positionné lit tout : le login et le mot de passe, chaque commande tapée, et toute la sortie, et peut détourner la session TCP en direct pour injecter des commandes en tant qu'utilisateur authentifié. L'interception est souvent l'attaque Telnet la plus facile là où une position réseau existe, sans avoir à deviner d'identifiants."
keywords:
  - interception telnet
  - clair
  - sniffing de mot de passe
  - détournement de session
  - mitm
---

# Interception de trafic

Parce que Telnet ne chiffre rien, un attaquant disposant d'une position réseau obtient tout gratuitement : le nom d'utilisateur et le mot de passe tels qu'ils sont tapés au login, chaque commande que l'utilisateur exécute, et toute la sortie du serveur. Au-delà de la capture passive, la session TCP non authentifiée et non chiffrée peut être détournée, en injectant des commandes qui s'exécutent en tant qu'utilisateur déjà authentifié. Là où une position de capture ou on-path existe, l'interception est généralement l'attaque Telnet la plus facile et la plus discrète, contournant entièrement l'authentification.

```bash
# capture Telnet from an on-path position
tcpdump -i eth0 -A port 23 -w telnet.pcap
# follow the TCP stream to read credentials, commands, and output
```

## Sous-thèmes

- **[Sniffing de mot de passe](password-sniffing.md)** : capturer les identifiants depuis le login en clair.
- **[Interception de commandes](command-interception.md)** : lire les commandes et la sortie en transit.
- **[Détournement de session](session-hijacking.md)** : prendre le contrôle de la session Telnet en direct.

## Références

- [RFC 854 (Telnet, sans chiffrement)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
