---
title: "Divulgation de hachage RAKP IPMI : récupération de hachage de mot de passe pré-authentification"
order: 2
description: "L'échange d'authentification RAKP d'IPMI 2.0 renvoie un HMAC salé du mot de passe de l'utilisateur demandé avant que l'authentification ne soit terminée, de par sa conception. Un attaquant demande l'échange pour un nom d'utilisateur et reçoit un hachage cassable, de sorte que tout BMC parlant IPMI 2.0 divulgue un hachage de mot de passe pour chaque compte à un attaquant non authentifié aux fins de cassage hors ligne."
keywords:
  - rakp
  - ipmi 2.0
  - divulgation de hachage
  - cassage hors ligne
  - bmc
---

# Divulgation de hachage RAKP IPMI

La faille ici réside dans la spécification IPMI 2.0 elle-même, et non dans une implémentation donnée. La poignée de main RAKP (RMCP+ Authenticated Key-Exchange Protocol) fait que le BMC renvoie, dans le message RAKP 2, un HMAC salé calculé sur le mot de passe de l'utilisateur demandé, avant que le client n'ait rien prouvé. Ainsi, un attaquant qui initie l'échange pour un nom d'utilisateur reçoit un hachage du mot de passe de ce compte et le casse hors ligne. Comme cela est imposé par la spécification, pratiquement tout BMC parlant IPMI 2.0 est affecté : un attaquant non authentifié récupère un hachage de mot de passe cassable pour root et tout autre compte, et les mots de passe de BMC faibles (courants) tombent rapidement.

```bash
# retrieve the RAKP hash for a user and crack it offline
#   Metasploit module automates the RAKP exchange and outputs a hash
#   auxiliary/scanner/ipmi/ipmi_dumphashes  (RHOSTS, USER_FILE)
# the output is a crackable format:
#   <user>:<16-byte salt/exchange data>:<HMAC>  -> feed to hashcat mode 7300 (IPMI2 RAKP)
hashcat -m 7300 ipmi_rakp.hashes wordlist.txt
```

## Notes d'exploitation

- C'est pré-authentification et au niveau de la spécification, donc cela s'applique à presque tous les BMC IPMI 2.0 ; l'attaquant n'a besoin que de connaître ou de deviner les noms d'utilisateur (`root`, `ADMIN`, `admin` sont quasi universels) pour demander leurs hachages.
- La valeur récupérée est un HMAC salé cassable hors ligne (hashcat mode 7300) ; les mots de passe de BMC sont fréquemment des défauts ou faibles, de sorte que le cassage réussit couramment.
- Un mot de passe de BMC cassé donne le contrôle administratif authentifié du contrôleur (alimentation, console, média virtuel), le même impact que le contournement cipher 0 mais via un identifiant récupéré.
- Énumérez d'abord les noms d'utilisateur (les défauts et tout nom divulgué) ; combinez avec les [identifiants par défaut des BMC](bmc-default-credentials.md) (le mot de passe cassé est de toute façon souvent un défaut) et le [cipher 0](ipmi-cipher-0-bypass.md).

## Outils

- [Metasploit ipmi_dumphashes](https://www.metasploit.com/)
- [hashcat (mode 7300, IPMI2 RAKP)](https://hashcat.net/hashcat/)

## Références

- [Dan Farmer: IPMI RAKP hash disclosure](http://fish2.com/ipmi/remote-pw-cracking.html)
- [IPMI 2.0 RAKP](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
