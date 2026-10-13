---
title: "Interception de commandes : lire les commandes et la sortie Telnet en transit"
description: "Au-delà du login, tout le contenu d'une session Telnet, les commandes que l'utilisateur exécute et la sortie du serveur, traverse le réseau en clair. Un attaquant positionné reconstruit la session complète à partir du trafic, capturant la configuration, les identifiants secondaires saisis pendant la session, les résultats de commandes et les données sensibles que l'utilisateur consulte."
keywords:
  - interception de commandes
  - capture de session
  - clair
  - divulgation de sortie
  - telnet
---

# Interception de commandes

Le sniffing de mot de passe capture le login ; l'interception de commandes capture tout ce qui vient après. Puisque toute la session Telnet est en clair, un attaquant sur le chemin reconstruit l'interaction complète : chaque commande que l'utilisateur exécute, toute la sortie du serveur, tout identifiant secondaire tapé pendant la session (un mot de passe `enable` sur un routeur, un mot de passe `su`/`sudo`, des identifiants pour un autre système), et toutes les données que l'utilisateur consulte. Cela donne souvent plus que le login lui-même, parce que les opérateurs exécutent des commandes privilégiées et saisissent d'autres secrets à l'intérieur de la session.

```bash
# capture and reconstruct the full session
tcpdump -i eth0 -A port 23 -w telnet.pcap
# Wireshark Follow TCP Stream shows the entire session (commands + output)
tshark -r telnet.pcap -q -z follow,tcp,ascii,0
```

## Notes d'exploitation

- La capture de session dépasse fréquemment le login en valeur : mots de passe `enable`/`su`/`sudo` saisis en cours de session, identifiants pour d'autres systèmes, configuration d'appareil (y compris des secrets dans un `show running-config`), et toute sortie sensible apparaissent en clair.
- Reconstruisez le flux bidirectionnel complet pour associer les commandes à leur sortie ; la direction client vers serveur contient ce que l'utilisateur a tapé, la direction serveur vers client contient les résultats.
- C'est passif et discret ; combinez avec le [sniffing de mot de passe](password-sniffing.md) à partir de la même capture pour le login plus tout ce qui suit.
- Pour un contrôle actif plutôt qu'une observation, passez au [détournement de session](session-hijacking.md).

## Références

- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
