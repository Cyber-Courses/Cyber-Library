---
order: 4
title: "Exposition d'identifiants et de secrets : clés et identifiants dans les OID SNMP"
description: "Les agents SNMP exposent des secrets aussi bien dans les OID standard que propriétaires : d'autres community strings et des données d'utilisateurs SNMPv3, des clés sans-fil (WEP/WPA) et des clés pré-partagées VPN sur les équipements réseau, ainsi que des identifiants ou jetons que les applications placent dans des OID d'entreprise personnalisés. Un walk en lecture récupère fréquemment du matériel qui déverrouille l'équipement et d'autres, au-delà de la compromission de l'équipement lui-même."
keywords:
  - identifiants snmp
  - clés wep wpa
  - psk vpn
  - oid personnalisé
  - exposition de secrets
---

# Exposition d'identifiants et de secrets

Au-delà de la topologie et des données d'hôte, SNMP expose fréquemment de véritables secrets. Les équipements réseau stockent du matériel sensible dans leurs MIB d'entreprise, lisible via SNMP : les points d'accès sans-fil exposent les clés WEP/WPA et les SSID, les extrémités VPN exposent les clés pré-partagées et la configuration, et de nombreux équipements exposent leurs autres community strings SNMP (potentiellement plus solides) et les paramètres d'utilisateurs SNMPv3, de sorte qu'une string de lecture faible peut divulguer les meilleurs identifiants. Les agents des éditeurs et des applications placent aussi des données dans des OID d'entreprise personnalisés (`1.3.6.1.4.1.<vendor>`) que les développeurs ne considéraient pas comme secrets, incluant parfois des identifiants, des jetons ou des URL internes. Parcourir l'arbre complet, en particulier les sous-arbres d'entreprise, en prêtant attention aux valeurs de type chaîne, est la manière de les trouver.

```bash
# walk the vendor enterprise subtree and grep for secret-looking values
snmpbulkwalk -v2c -c public -Oqv <target> 1.3.6.1.4.1 \
  | grep -iE 'pass|pwd|secret|key|psk|wpa|wep|community|token|ssid' | head
# wireless gear: SSIDs and keys live in the vendor WLAN MIB (subtree is vendor-specific)
# the config-copy route on Cisco yields all device credentials at once (see Cisco page)
```

## Notes d'exploitation

- Le sous-arbre d'entreprise de l'éditeur (`1.3.6.1.4.1`) est l'endroit où vivent les secrets propres au produit ; videz-le et recherchez par motif les valeurs de type chaîne au contenu ressemblant à des identifiants, car les développeurs y exposent souvent des données sans les traiter comme sensibles.
- Une string de lecture faible qui expose les autres community strings de l'équipement ou ses utilisateurs SNMPv3 constitue une élévation de privilège : utilisez la string ou les identifiants plus forts ainsi divulgués pour un accès en écriture ou pour des équipements que la string faible ne couvre pas.
- Les clés sans-fil et VPN récupérées ici sont réutilisables pour rejoindre le réseau ou déchiffrer le trafic ; les identifiants de gestion de l'équipement permettent de pivoter vers les autres interfaces de l'équipement (SSH/HTTP).
- Sur Cisco, l'ensemble complet des identifiants provient de la configuration elle-même ; voir [Exfiltration de configuration Cisco](cisco-configuration-exfiltration.md), que cette page complète pour les agents non-Cisco et applicatifs.

## Références

- [HackTricks: SNMP secrets](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [Enterprise OID registry (IANA PEN)](https://www.iana.org/assignments/enterprise-numbers/)
