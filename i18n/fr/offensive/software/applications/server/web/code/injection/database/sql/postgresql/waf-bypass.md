---
order: 9
title: "Contournement de WAF et de filtre pour l'injection PostgreSQL"
description: "Évader les filtres dans l'injection PostgreSQL avec la construction de chaînes CHR(), le dollar-quoting, la concaténation et les alternatives pg_catalog aux noms bloqués."
keywords:
  - contournement de WAF
  - fonction CHR
  - dollar quoting
  - évasion de filtre PostgreSQL
  - alternative pg_catalog
---

# Contournement de WAF

Le dialecte de PostgreSQL offre plusieurs moyens de contourner les filtres de signatures qui bloquent les guillemets, les mots-clés ou la chaîne `information_schema`.

Les guillemets s'évitent en construisant des chaînes à partir de codes de caractères avec `CHR()` joints par `||`, ou avec le dollar-quoting, qui n'a besoin d'aucun caractère guillemet du tout :

```sql
-- '/etc/passwd' without quotes
CHR(47)||CHR(101)||CHR(116)||CHR(99)||CHR(47)||CHR(112)||CHR(97)||CHR(115)||CHR(115)||CHR(119)||CHR(100)
-- dollar-quoted literal
$$/etc/passwd$$
```

Les signatures de mots-clés se fractionnent avec la concaténation et les commentaires inline, puisque PostgreSQL ignore `/**/` entre les jetons :

```sql
' UNION/**/SELECT/**/NULL,version(),NULL-- 
```

Quand `information_schema` est sur liste de blocage, le `pg_catalog` natif atteint les mêmes données sous d'autres noms : `pg_tables` pour les tables, `pg_namespace` pour les schémas, et `pg_class` joint à `pg_attribute` pour les colonnes. Quand `version()` est filtré, `current_setting('server_version')` renvoie la chaîne de version, et `current_setting('server_version_num')` la renvoie sous forme de nombre.

Les astuces numériques et d'encodage aident aussi : `CHR()` et `convert_from(decode('...','hex'),'UTF8')` reconstruisent les chaînes filtrées, et caster avec `::text` évite les mots-clés `CAST` explicites. Comme avec d'autres moteurs, l'approche consiste à atteindre le même objet ou la même valeur via un synonyme que le filtre ne connaît pas, plutôt qu'à vaincre le filtre de front.

## Outils

- **sqlmap** : scripts tamper (`charencode`, `chardoubleencode`) pour l'évasion de filtre et de WAF.
- **Burp Repeater** : composer à la main les charges de contournement `CHR()` et dollar-quoted.

## Références

- PostgreSQL Documentation: `chr`, dollar-quoted strings, `current_setting`, system catalogs
- OWASP Testing Guide: Testing for SQL Injection
