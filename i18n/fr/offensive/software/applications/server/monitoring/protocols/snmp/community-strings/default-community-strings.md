---
title: "Chaînes de communauté par défaut : les valeurs par défaut SNMP quasi universelles"
description: "Les agents SNMP sont livrés avec des chaînes de communauté par défaut, massivement public en lecture et private en lecture-écriture, plus des valeurs par défaut propres aux fabricants. Elles sont rarement modifiées, en particulier sur les imprimantes, commutateurs et appareils, de sorte qu'essayer les valeurs par défaut documentées est la première et la plus productive façon d'obtenir un accès SNMP."
keywords:
  - default community
  - public
  - private
  - vendor default
  - snmp
---

# Chaînes de communauté par défaut

Les valeurs par défaut de SNMP sont la voie d'entrée la plus fiable. L'écrasante majorité des agents sont livrés avec `public` comme chaîne en lecture seule et `private` (ou `write`) comme chaîne en lecture-écriture, et celles-ci sont laissées inchangées sur une large part des équipements réels, imprimantes, commutateurs, points d'accès, caméras IP et appareils en particulier. Les fabricants ajoutent aussi leurs propres valeurs par défaut (par exemple `cisco`, `ILMI`, `admin`, `security` sur divers matériels). La première action contre tout agent SNMP est donc d'essayer les valeurs par défaut documentées pour la lecture et, séparément, pour l'écriture, car une chaîne de lecture par défaut donne une énumération complète et une chaîne d'écriture par défaut donne le contrôle de l'équipement.

```bash
# test common read defaults
for c in public private cisco ILMI community manager admin security read; do
  snmpget -v2c -c $c -t1 -r0 <target> 1.3.6.1.2.1.1.1.0 2>/dev/null \
    | sed "s/^/[$c] /"; done
# specifically test write (needs a read-write string): a harmless re-set of sysContact
snmpset -v2c -c private <target> 1.3.6.1.2.1.1.4.0 s "$(snmpget -v2c -c private -Ov -Oq <target> 1.3.6.1.2.1.1.4.0 2>/dev/null)" 2>/dev/null && echo "WRITE works: private"
```

## Notes d'exploitation

- Essayez séparément les valeurs par défaut de lecture et d'écriture : `public` pour la lecture est quasi universel, et un `private` fonctionnel pour l'écriture donne le contrôle de l'équipement ; de nombreux appareils ont l'une des valeurs par défaut mais pas l'autre.
- Identifiez d'abord l'équipement lorsque c'est possible ([Informations sur l'équipement](../enumeration/device-information.md)) pour ajouter à la liste les valeurs par défaut propres au fabricant de ce produit.
- Les appareils et équipements embarqués (imprimantes, caméras, PDU, commutateurs) sont les plus susceptibles de conserver les valeurs par défaut et sont de grande valeur car SNMP expose souvent leur configuration complète.
- Si les valeurs par défaut échouent, passez aux [chaînes faibles](weak-community-strings.md) et au [brute force](brute-force.md) ; une chaîne trouvée pilote ensuite l'[énumération](../enumeration/index.md) et, si elle est en lecture-écriture, l'[accès en écriture](../write-access.md).

## Références

- [SecLists : common SNMP community strings](https://github.com/danielmiessler/SecLists/blob/master/Discovery/SNMP/common-snmp-community-strings.txt)
- [HackTricks : SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
