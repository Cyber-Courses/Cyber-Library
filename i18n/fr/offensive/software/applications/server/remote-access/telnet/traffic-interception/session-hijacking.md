---
title: "Détournement de session : prendre le contrôle d'une session Telnet en direct"
description: "Une session Telnet est un flux TCP non chiffré et non authentifié après le login, de sorte qu'un attaquant on-path y injecte des commandes qui s'exécutent en tant qu'utilisateur déjà authentifié, ou prend entièrement le contrôle de la session. Cela contourne complètement l'authentification en chevauchant une session établie, s'exécutant avec les privilèges de la victime sur la cible."
keywords:
  - détournement de session
  - injection tcp
  - telnet
  - injection de commandes
  - mitm
---

# Détournement de session

Telnet n'authentifie qu'au login ; ensuite la session n'est qu'un flux TCP non chiffré sans authentification par paquet. Un attaquant on-path la détourne donc : en injectant des segments TCP forgés avec les bons numéros de séquence (observables parce que le trafic est en clair), il insère des commandes dans le flux que le serveur exécute en tant qu'utilisateur déjà authentifié, ou désynchronise et prend entièrement le contrôle de la session. Cela contourne complètement le login, l'attaquant n'a jamais besoin du mot de passe, il chevauche une session que quelqu'un d'autre a authentifiée, et s'exécute avec les privilèges de cet utilisateur.

```bash
# on-path position to the Telnet session (ARP/route)
# observe the stream to learn the TCP sequence/ack state (cleartext makes this easy)
# inject a command segment with the right seq/ack so the server executes it as the user
# classic tooling automated TCP session hijacking against telnet/rlogin:
#   (e.g. the historic Hunt/Juggernaut tools; modern equivalents craft segments directly)
```

## Notes d'exploitation

- Le détournement chevauche une session authentifiée, il n'a donc besoin d'aucun identifiant, seulement d'une position on-path et du flux en clair pour lire l'état séquence/ack destiné à l'injection.
- Injecter une seule commande (ajouter un utilisateur, écrire une clé, démarrer un reverse shell) suffit souvent et est moins perturbateur qu'une prise de contrôle complète, qui peut désynchroniser et alerter l'utilisateur.
- La commande injectée s'exécute avec les privilèges de la victime, fréquemment administratifs sur l'appareil, de sorte qu'une seule commande injectée peut être une compromission totale.
- C'est le versant actif de l'interception Telnet ; les équivalents passifs sont le [sniffing de mot de passe](password-sniffing.md) et l'[interception de commandes](command-interception.md), et la même faiblesse en clair rend les trois possibles.

## Références

- [HackTricks: Telnet session hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [Détournement de session TCP (technique classique)](https://www.rfc-editor.org/rfc/rfc793)
