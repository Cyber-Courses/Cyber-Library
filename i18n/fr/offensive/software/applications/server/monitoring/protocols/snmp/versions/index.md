---
title: "Versions : faiblesses des versions de SNMP"
order: 3
description: "SNMP compte trois versions à la sécurité très différente. v1 et v2c s'authentifient avec une community string en clair et sans chiffrement, elles sont donc interceptables et abusées trivialement. v3 ajoute une authentification et une confidentialité basées sur l'utilisateur, mais elle divulgue encore les noms d'utilisateurs avant l'authentification et peut être attaquée hors ligne ou dégradée. Détecter la version détermine l'attaque."
keywords:
  - snmp version
  - snmpv1
  - snmpv2c
  - snmpv3
  - version detection
---

# Versions

Les trois versions de SNMP diffèrent fondamentalement en matière de sécurité, de sorte qu'identifier laquelle un agent parle détermine toute l'attaque. SNMPv1 et v2c partagent le même modèle faible : la community string est le seul identifiant, elle est envoyée en clair, et rien n'est chiffré, donc la chaîne se capture par interception et tout est exposé à quiconque la possède. SNMPv3 est un protocole différent doté d'une sécurité basée sur l'utilisateur, avec authentification et confidentialité (chiffrement) optionnelles, ce qui supprime la faiblesse de la chaîne en clair, mais il divulgue encore les noms d'utilisateurs pendant le handshake de découverte du moteur et reste exposé au cassage hors ligne d'une authentification capturée ainsi qu'au downgrade lorsque des versions plus faibles restent activées. De nombreux agents répondent à plusieurs versions à la fois.

## Sous-sujets

- **[Détection de version](version-detection.md)** : déterminer quelles versions un agent accepte.
- **[SNMPv1 et v2c en clair](snmpv1-and-v2c-cleartext.md)** : la faiblesse de la community string en clair.
- **[Attaques SNMPv3](snmpv3-attacks.md)** : énumération des noms d'utilisateurs, cassage hors ligne et downgrade.

## Références

- [RFC 3414 (SNMPv3 USM)](https://datatracker.ietf.org/doc/html/rfc3414)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
