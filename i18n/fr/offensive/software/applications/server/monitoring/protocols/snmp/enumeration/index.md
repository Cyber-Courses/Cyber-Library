---
title: "Énumération : parcourir un agent SNMP"
order: 2
description: "L'énumération SNMP lit l'arbre d'OID de l'agent à l'aide d'une community string valide : le groupe system de base pour l'identité de l'équipement, puis un parcours complet de la MIB pour extraire tout ce que l'agent expose. C'est le parcours qui apporte la valeur : un bulk walking efficace et le ciblage des bons sous-arbres fournisseur transforment une chaîne de lecture en une image complète de l'équipement."
keywords:
  - snmp enumeration
  - snmpwalk
  - snmpbulkwalk
  - mib
  - oid
---

# Énumération

L'énumération est l'action SNMP centrale : avec une community string de lecture fonctionnelle, on parcourt l'arbre d'OID de l'agent et on lit ce qu'il expose. Deux niveaux comptent. Le groupe system de base (`1.3.6.1.2.1.1`) fournit l'identité de l'équipement, une description, un nom, un emplacement, un contact et un uptime, de quoi établir l'empreinte de l'équipement. Le parcours complet extrait ensuite tout ce que l'agent publie, ce qui, sur un équipement réel, est considérable : interfaces, adresses, routes, tables ARP, processus, logiciels et données propres au fournisseur. C'est le parcours qui rapporte, donc le réaliser efficacement (opérations en bloc) et savoir quels sous-arbres cibler constitue le savoir-faire.

```bash
# basic identity, then the full tree
snmpget -v2c -c public <target> 1.3.6.1.2.1.1.1.0        # sysDescr (one OID)
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.1           # system group
snmpbulkwalk -v2c -c public <target>                     # full walk, far faster than snmpwalk
snmp-check -c public <target>                            # formatted summary of common data
```

## Sous-sujets

- **[Informations sur l'équipement](device-information.md)** : le groupe system et l'identité de l'équipement.
- **[Parcours de MIB et d'OID](mib-and-oid-walking.md)** : extraire efficacement l'arbre complet.

## Références

- [net-snmp snmpwalk/snmpbulkwalk](http://www.net-snmp.org/docs/man/snmpbulkwalk.html)
- [HackTricks: SNMP enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
