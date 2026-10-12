---
title: "SNMP : attaquer le Simple Network Management Protocol"
order: 1
description: "SNMP sur UDP 161 expose l'intégralité de l'état d'un équipement à travers un arbre d'OID, protégé seulement par une community string sur v1 et v2c. La surface d'attaque consiste à obtenir cette chaîne (par défaut, faible, par force brute), à parcourir l'agent pour divulguer des données système, réseau et d'identifiants, à exfiltrer la configuration de l'équipement et, avec une chaîne en lecture-écriture, à reconfigurer l'équipement."
keywords:
  - snmp
  - udp 161
  - community string
  - oid
  - mib
---

# SNMP

SNMP (Simple Network Management Protocol) fonctionne sur UDP 161 et expose l'état d'un équipement géré sous la forme d'un arbre d'identifiants d'objets (OID) organisés en MIB. Sur SNMPv1 et v2c, toujours dominants, le seul contrôle d'accès est une community string envoyée en clair, une chaîne de lecture (par convention `public`) pour les requêtes et une chaîne de lecture-écriture (par convention `private`) pour modifier les paramètres. Ce modèle faible fait de SNMP l'une des surfaces de reconnaissance et d'exposition d'identifiants les plus productives d'un réseau : routeurs, commutateurs, pare-feu, imprimantes, serveurs et appliances y répondent tous, et ce qu'ils renvoient comprend l'inventaire système, la topologie réseau et, fréquemment, des identifiants et des configurations complètes d'équipements. Avec une chaîne de lecture-écriture, SNMP permet aussi de reconfigurer l'équipement.

```bash
# detect and version the agent
nmap -sU -p161 -sV --script snmp-info <target>
snmpwalk -v2c -c public <target> 2>/dev/null | head     # does a read string work?
```

## Sous-sujets

- **[Énumération](enumeration/index.md)** : parcourir l'agent et lire les données de l'équipement.
- **[Community strings](community-strings/index.md)** : obtenir des chaînes de lecture et de lecture-écriture.
- **[Divulgation d'informations](information-disclosure/index.md)** : exposition du système, de la topologie, de la configuration et des identifiants.
- **[Versions](versions/index.md)** : faiblesses en clair de v1/v2c et attaque de v3.
- **[Accès en écriture](write-access.md)** : reconfigurer l'équipement avec une chaîne de lecture-écriture.

## Références

- [RFC 3416 (SNMP protocol operations)](https://datatracker.ietf.org/doc/html/rfc3416)
- [HackTricks: SNMP (161)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [net-snmp tools](http://www.net-snmp.org/)
