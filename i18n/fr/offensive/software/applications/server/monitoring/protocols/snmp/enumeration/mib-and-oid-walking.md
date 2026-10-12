---
title: "Parcours de MIB et d'OID : extraire l'arbre SNMP complet"
description: "Un parcours SNMP complet lit chaque OID que l'agent expose, bien plus que le groupe system : interfaces, adresses, tables de routage et ARP, et sous-arbres fournisseur. snmpbulkwalk le récupère efficacement grâce à GETBULK, et traduire les OID numériques en noms à l'aide des MIB chargées rend l'extraction lisible, transformant une chaîne de lecture en l'état publié complet de l'équipement."
keywords:
  - snmpwalk
  - snmpbulkwalk
  - getbulk
  - mib
  - snmptranslate
---

# Parcours de MIB et d'OID

Au-delà du groupe system, l'agent publie un arbre volumineux, et le parcourir est la façon d'en extraire le contenu réel. `snmpwalk` émet des requêtes GETNEXT successives à partir d'un OID de départ jusqu'à la fin du sous-arbre ; `snmpbulkwalk` utilise l'opération GETBULK de v2c pour récupérer de nombreuses valeurs par requête, ce qui est radicalement plus rapide face à un équipement comptant des milliers d'OID et constitue le choix par défaut en pratique. Les OID numériques sont opaques, donc charger les MIB et traduire les noms rend la sortie lisible et révèle quels sous-arbres comptent. Les sous-arbres standard de MIB-II fournissent à eux seuls les interfaces (`1.3.6.1.2.1.2`), l'adressage et le routage IP (`1.3.6.1.2.1.4`) ainsi que les tables de connexions TCP/UDP ; les sous-arbres entreprise des fournisseurs (`1.3.6.1.4.1`) contiennent les données propres au produit.

```bash
# fast full walk (GETBULK); start points for the useful subtrees
snmpbulkwalk -v2c -c public <target>                     # everything
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.2       # interfaces (ifTable)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.21    # ipRouteTable (routing)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.22    # ipNetToMediaTable (ARP)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.4.1         # vendor enterprise subtree
# make it readable: load all MIBs and translate
snmpbulkwalk -v2c -c public -m ALL <target> | head
snmptranslate -On SNMPv2-MIB::sysDescr.0                 # name <-> numeric OID
```

## Notes d'exploitation

- Utilisez `snmpbulkwalk` (GETBULK) et non `snmpwalk` pour une extraction complète ; sur les grands équipements la différence se compte en minutes contre une heure, et certains agents limitent le débit de la boucle GETNEXT plus lente.
- Ciblez par objectif plutôt que d'extraire à l'aveugle lorsque l'équipement est volumineux : `ifTable` et `ipRouteTable`/`ipNetToMediaTable` donnent l'image du réseau, le sous-arbre fournisseur `1.3.6.1.4.1` contient la configuration et les identifiants, et la host-resources MIB (`1.3.6.1.2.1.25`) donne les processus et les logiciels sur les serveurs.
- Charger les MIB (`-m ALL`, ou les MIB du fournisseur) traduit les OID numériques en noms pour que la sortie soit interprétable ; `snmptranslate` établit la correspondance entre les deux.
- Le parcours alimente les pages de divulgation : la topologie à partir des tables réseau, et le butin plus riche sous [Divulgation d'informations](../information-disclosure/index.md).

## Références

- [net-snmp snmpbulkwalk](http://www.net-snmp.org/docs/man/snmpbulkwalk.html)
- [RFC 3416 (GETBULK)](https://datatracker.ietf.org/doc/html/rfc3416)
