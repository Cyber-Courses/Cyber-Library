---
order: 6
title: "Contournement de WAF et de filtres pour l'injection IBM Db2"
description: "Contourner les filtres dans l'injection IBM Db2 avec la construction de chaînes CHR() et CONCAT, l'hexadécimal via CAST, les commentaires en ligne et la variation de casse."
keywords:
  - contournement de WAF
  - fonction CHR
  - CONCAT
  - littéral hexadécimal
  - évasion de filtre Db2
---

# Contournement de WAF

Le dialecte de Db2 offre les moyens habituels de franchir les filtres de signatures qui bloquent les guillemets ou les mots-clés.

Les chaînes sont construites sans guillemets à partir de codes de caractères avec `CHR()` joints par `||` ou `CONCAT()`, ce qui déjoue les filtres de guillemets :

```sql
-- 'USERS' without quotes
CHR(85)||CHR(83)||CHR(69)||CHR(82)||CHR(83)
```

L'hexadécimal peut fournir des octets via un cast, évitant les littéraux entre guillemets à certaines positions :

```sql
CAST(x'5553455253' AS VARCHAR(5))
```

Les mots-clés SQL sont fractionnés avec des commentaires en ligne `/**/` (qui remplacent les espaces) et avec la variation de casse, et non par concaténation : `||`/`CONCAT` construisent des valeurs de chaîne à l'intérieur des expressions, et le résultat n'est pas réanalysé comme un mot-clé, donc ils ne peuvent pas reconstituer `UNION` ou `SELECT`. Utilisez les commentaires et la casse pour les mots-clés :

```sql
' UnIoN/**/SeLeCt/**/TABNAME/**/FROM/**/SYSCAT.TABLES-- 
```

La concaténation déjoue plutôt les filtres sur les valeurs de littéraux de chaîne (un nom de table, une chaîne de payload), reconstituant une sous-chaîne bloquée à partir de morceaux avec `CONCAT`, `CHR()`, `TRANSLATE` ou `REPLACE`. Les registres spéciaux (`CURRENT USER`, `CURRENT SERVER`) fournissent aussi des données sans syntaxe d'appel de fonction qu'un filtre pourrait signaler.

Comme avec les autres moteurs, le but est d'exprimer la même requête via des synonymes et des encodages que le filtre ne reconnaît pas plutôt que de l'affronter frontalement, en combinant plusieurs de ces techniques là où un filtre bloque plus d'un schéma. Notez qu'un littéral hexadécimal Db2 (`x'...'`) utilise toujours des guillemets, donc pour une construction véritablement sans guillemets, reposez-vous sur la concaténation de `CHR()`.

## Outils

- **sqlmap** : les scripts tamper automatisent les réécritures `CHR()`, de commentaires et de casse.
- **ghauri** : évasion de WAF intégrée pour les payloads Db2.
- **Burp Repeater** : ajustez à la main les encodages et les registres spéciaux jusqu'à contourner le filtre.

## Références

- IBM Db2 SQL Reference : CHR, CONCAT, CAST, comments, special registers
- OWASP Testing Guide : Testing for SQL Injection
