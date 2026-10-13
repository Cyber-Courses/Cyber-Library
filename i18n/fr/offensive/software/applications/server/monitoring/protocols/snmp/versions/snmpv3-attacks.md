---
title: "Attaques SNMPv3 : énumération des noms d'utilisateurs, cassage et downgrade"
order: 3
description: "SNMPv3 ajoute une authentification des utilisateurs et une confidentialité, mais il fuit encore : l'échange de découverte du moteur et de report confirme si un nom d'utilisateur est valide avant l'authentification, ce qui permet l'énumération des utilisateurs, et un échange authentifié capturé se casse hors ligne pour les mots de passe d'authentification faibles. Là où v1/v2c reste activée, l'attaque la plus simple consiste à y revenir entièrement par downgrade."
keywords:
  - snmpv3
  - username enumeration
  - usm
  - offline cracking
  - downgrade
---

# Attaques SNMPv3

SNMPv3 remplace la community string par une sécurité basée sur l'utilisateur (USM) : un nom d'utilisateur, une clé d'authentification (MD5 ou SHA) et, en option, une clé de confidentialité pour le chiffrement. Cela supprime la faiblesse de la chaîne en clair, mais SNMPv3 reste attaquable. D'abord, l'énumération des noms d'utilisateurs : le handshake de découverte du moteur et les PDU de report que l'agent renvoie distinguent un nom d'utilisateur valide d'un nom invalide avant la fin de l'authentification, de sorte qu'un attaquant confirme les noms d'utilisateurs réels en sondant. Ensuite, le cassage hors ligne : une requête authentifiée capturée contient le HMAC sur des données connues, donc les mots de passe d'authentification faibles sont récupérés hors ligne. Enfin, et c'est le plus simple, le downgrade : si l'équipement répond aussi encore à v1/v2c (courant), ignorez v3 et attaquez le chemin en clair.

```bash
# username enumeration: valid vs invalid users elicit different engine/report responses
snmpget -v3 -l authNoPriv -u <guess> -a SHA -A badpass <target> 1.3.6.1.2.1.1.1.0 2>&1
#   "Unknown user name" vs "Authentication failure" distinguishes invalid vs valid user
# tools automate the enumeration (e.g. Metasploit snmp_login / snmpv3 enum modules)
# offline cracking of captured v3 auth (weak MD5/SHA auth password)
#   capture an authenticated request, extract the USM fields, crack with a wordlist
# downgrade: if v1/v2c also answer, attack them instead (see version detection)
```

## Notes d'exploitation

- Les réponses différentes de l'agent à un nom d'utilisateur inconnu par rapport à un nom connu avec un mauvais mot de passe constituent l'oracle d'énumération ; un nom d'utilisateur validé est la cible du cassage hors ligne ou de la devinette du mot de passe d'authentification.
- Le cassage hors ligne fonctionne contre les phrases secrètes d'authentification faibles (le HMAC porte sur un texte clair connu) ; des clés d'authentification/confidentialité robustes y résistent, donc le succès dépend de la force du mot de passe.
- Le downgrade est généralement la voie de moindre résistance : les entreprises laissent v1/v2c activée pour les outils hérités, donc confirmez les versions ([Détection de version](version-detection.md)) et préférez la voie en clair lorsqu'elle est disponible.
- Un utilisateur v3 récupéré avec authentification (et confidentialité) accorde une lecture complète et, selon la vue de l'utilisateur, l'écriture ; traitez-le comme n'importe quel identifiant d'équipement.

## Références

- [RFC 3414 (SNMPv3 USM)](https://datatracker.ietf.org/doc/html/rfc3414)
- [HackTricks: SNMPv3](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
