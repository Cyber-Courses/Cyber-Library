---
title: "Chaînes de communauté : obtenir un accès SNMP"
order: 1
description: "Sur SNMPv1 et v2c, la chaîne de communauté est le seul identifiant : une chaîne de lecture autorise l'interrogation, une chaîne de lecture-écriture autorise la reconfiguration. Les chaînes s'obtiennent de la même façon que tous les identifiants faibles, en essayant les valeurs par défaut quasi universelles, en devinant des valeurs faibles ou prévisibles, et en les cassant par force brute avec une liste de mots, et une chaîne fonctionnelle constitue tout l'accès."
keywords:
  - community string
  - public
  - private
  - default
  - brute force
---

# Chaînes de communauté

Sur SNMPv1 et v2c, la chaîne de communauté constitue l'intégralité du contrôle d'accès : il n'y a pas de nom d'utilisateur, et la chaîne circule en clair. Par convention, `public` est la chaîne en lecture seule et `private` la chaîne en lecture-écriture, mais n'importe quelle valeur peut être configurée. Obtenir une chaîne est donc l'objectif, et elle s'obtient comme tout identifiant faible : en essayant les valeurs par défaut quasi universelles, en devinant des valeurs faibles ou prévisibles (souvent dérivées de l'organisation ou du nom d'hôte), et en les cassant par force brute contre une liste de mots. Une chaîne de lecture donne une énumération et une divulgation complètes ; une chaîne de lecture-écriture donne en plus la reconfiguration. L'accès en lecture et en écriture peut utiliser des chaînes différentes, de sorte que trouver une chaîne de lecture n'implique pas l'écriture, et inversement.

```bash
# quickest: test the obvious strings
for c in public private community manager; do
  snmpget -v2c -c $c -t1 -r0 <target> 1.3.6.1.2.1.1.1.0 2>/dev/null && echo "WORKS: $c"; done
```

## Sous-thèmes

- **[Chaînes de communauté par défaut](default-community-strings.md)** : les valeurs par défaut quasi universelles et propres aux fabricants.
- **[Chaînes de communauté faibles](weak-community-strings.md)** : chaînes prévisibles et dérivées de l'organisation.
- **[Brute force](brute-force.md)** : attaques par liste de mots contre la chaîne.

## Références

- [HackTricks : SNMP community strings](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [SecLists SNMP community strings](https://github.com/danielmiessler/SecLists)
