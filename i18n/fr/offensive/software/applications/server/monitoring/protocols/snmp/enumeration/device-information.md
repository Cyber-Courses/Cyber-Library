---
title: "Informations sur l'équipement : le groupe system de SNMP"
description: "Le groupe system de SNMP, à 1.3.6.1.2.1.1, renvoie la description, l'object ID, la durée de fonctionnement, le contact, le nom et l'emplacement d'un équipement. La chaîne sysDescr nomme généralement l'OS, le modèle et le firmware exacts, identifiant l'équipement pour la correspondance de vulnérabilités, et les champs de contact et d'emplacement divulguent des détails organisationnels utiles au ciblage et à l'ingénierie sociale."
keywords:
  - system group
  - sysdescr
  - sysname
  - syslocation
  - fingerprint
---

# Informations sur l'équipement

Le groupe system, OID `1.3.6.1.2.1.1`, est la première chose à lire : il renvoie un enregistrement d'identité compact et de grande valeur présent sur chaque agent SNMP. `sysDescr` (`.1.1.0`) est une description en texte libre qui nomme généralement le système d'exploitation, le modèle de l'équipement et la version du firmware ou du logiciel exacts, ce qui identifie précisément l'équipement pour la correspondance de vulnérabilités. `sysObjectID` (`.2.0`) identifie le fabricant et le produit par OID. `sysUpTime` (`.3.0`) donne la durée de fonctionnement (utile pour déduire la cadence de correctifs et le dernier redémarrage). Et `sysContact`, `sysName` et `sysLocation` (`.4`/`.5`/`.6`) sont des champs définis par l'administrateur qui divulguent fréquemment le nom et l'adresse e-mail d'un administrateur, le nom d'hôte de l'équipement et un emplacement physique.

```bash
# the whole system group in one walk
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.1
# or the individual high-value OIDs
snmpget -v2c -c public <target> \
  1.3.6.1.2.1.1.1.0 \   # sysDescr  (OS/model/firmware)
  1.3.6.1.2.1.1.4.0 \   # sysContact (admin name/email)
  1.3.6.1.2.1.1.5.0 \   # sysName    (hostname)
  1.3.6.1.2.1.1.6.0     # sysLocation
```

## Notes d'exploitation

- `sysDescr` est l'empreinte clé : elle indique généralement la plateforme et la version exactes (par exemple une branche Cisco IOS, un firmware d'imprimante, un build Windows), ce qui associe l'équipement à ses vulnérabilités connues et vous indique quels OID plus profonds (sous-arbres de fabricant) valent la peine d'être parcourus.
- `sysContact` et `sysLocation` sont du matériau pour la reconnaissance et l'ingénierie sociale : les e-mails d'administrateurs réels et les emplacements physiques sont courants, et le nom d'hôte dans `sysName` amorce une énumération supplémentaire.
- Ce groupe est lisible avec n'importe quelle chaîne de communauté de lecture et constitue la première étape la moins coûteuse ; il confirme aussi que la chaîne fonctionne avant un parcours complet.
- Associez la plateforme identifiée au bon suivi : Cisco à l'[exfiltration de configuration Cisco](../information-disclosure/cisco-configuration-exfiltration.md), Windows aux [informations sur l'hôte Windows](../information-disclosure/windows-host-information.md).

## Références

- [RFC 1213 (MIB-II system group)](https://datatracker.ietf.org/doc/html/rfc1213)
- [HackTricks : SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
