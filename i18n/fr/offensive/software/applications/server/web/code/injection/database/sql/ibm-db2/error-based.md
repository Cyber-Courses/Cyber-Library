---
order: 5
title: "Injection SQL basée sur les erreurs dans IBM Db2"
description: "Le canal d'erreur étroit d'IBM Db2 : utiliser des casts invalides et SIGNAL pour divulguer des valeurs quand c'est possible, et pourquoi l'inférence booléenne ou temporelle est la voie aveugle fiable."
keywords:
  - injection SQL basée sur les erreurs
  - erreurs Db2
  - cast invalide
  - SIGNAL SQLSTATE
  - SQLCODE
---

# Basée sur les erreurs

L'injection basée sur les erreurs est plus faible dans Db2 que dans PostgreSQL ou SQL Server, parce que les messages d'erreur de Db2 ne citent pas de façon fiable un résultat de requête arbitraire. Elle est surtout utile pour confirmer l'injection et faire le fingerprinting, les données réelles étant lues par inférence.

Une conversion invalide est l'erreur la plus simple. Caster une chaîne non numérique en nombre échoue avec une erreur de conversion de type `SQL0420N`/`SQL0180`, ce qui prouve que le point est actif :

```sql
' AND 1=CAST((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1) AS INT)-- 
```

Là où la valeur est non numérique, le cast échoue ; que la valeur fautive apparaisse dans le message retourné dépend du pilote et de la verbosité des erreurs, donc on ne peut pas s'y fier pour renvoyer des données arbitraires comme le font d'autres moteurs.

`SIGNAL` lève une erreur personnalisée dont le texte du message peut porter une valeur, ce qui est le plus proche de ce que Db2 offre comme canal de renvoi de valeur, bien que cela nécessite un contexte d'instruction composée ou de routine plutôt qu'une simple expression injectée. `MESSAGE_TEXT` accepte une valeur simple, pas un fullselect scalaire, donc le résultat est d'abord assigné à une variable déclarée et cette variable est fournie :

```sql
BEGIN DECLARE v VARCHAR(128); SET v = (SELECT CURRENT SERVER FROM SYSIBM.SYSDUMMY1); SIGNAL SQLSTATE '75001' SET MESSAGE_TEXT = v; END
```

Parce que la longueur du message est bornée et que le renvoi propre est inconstant, l'approche pratique contre Db2 est d'utiliser les erreurs pour confirmer et fingerprinter (un `SQLCODE`/`SQLSTATE` distinct prouve l'injection et peut lui-même servir d'oracle booléen), et d'extraire les données avec les techniques booléenne et temporelle, qui sont les canaux fiables ici.

## Outils

- **sqlmap** : détection et extraction automatisées (`--dbms=Db2`).
- **ghauri** : alternative rapide avec forte évasion de WAF.
- **db2** (ou clpplus) : le client Db2 pour reproduire les erreurs `CAST` et `SIGNAL`.

## Références

- IBM Db2 SQL Reference : CAST, SIGNAL, SQLSTATE and SQLCODE values
- OWASP Testing Guide : Testing for SQL Injection
