---
order: 4
title: "Évasion de filtres et de WAF pour l'injection SQLite"
description: "Contourner les filtres dans l'injection SQLite avec des commentaires inline, la construction de chaînes par char() et la concaténation pour passer outre les blocklists de mots-clés et de guillemets."
keywords:
  - contournement de WAF
  - fonction char
  - commentaires inline
  - concaténation
  - évasion de filtres SQLite
---

# Techniques d'évasion

La syntaxe réduite mais flexible de SQLite offre les moyens habituels de contourner les filtres de signatures.

Les chaînes sont construites sans guillemets à partir de codes de caractères avec `char()` joints par `||`, ce qui déjoue les filtres de guillemets :

```sql
-- 'users' without quotes
char(117)||char(115)||char(101)||char(114)||char(115)
```

Les commentaires inline `/**/` remplacent les espaces pour casser les signatures de tokens séparés par des espaces, et la concaténation découpe une chaîne de mot-clé bloquée :

```sql
' UNION/**/SELECT/**/name/**/FROM/**/sqlite_master-- 
```

Les mots-clés sont insensibles à la casse, donc la variation de casse (`UnIoN SeLeCt`) contourne les blocklists naïves sensibles à la casse. Les littéraux blob (`x'7573657273'`) aident contre les signatures de mots-clés et de valeurs de chaîne, mais notez qu'ils utilisent toujours des guillemets simples, donc contrairement au `0x...` sans guillemets de MySQL, ils ne déjouent pas un filtre de guillemets ; pour une construction réellement sans guillemets, utilisez la forme `char()` ci-dessus. Les conversions `CAST`/`unicode`/`char` reconstruisent des sous-chaînes filtrées.

Parce que SQLite est faiblement typé et possède un parseur minimal, il tolère aussi certaines bizarreries qui déroutent les filtres réglés pour des moteurs plus stricts, comme l'absence de clauses `FROM` (`UNION SELECT 1,2`) et des formes de littéraux flexibles. Comme pour les autres moteurs, l'objectif est d'exprimer la même requête au moyen de synonymes et d'encodages que le filtre ne reconnaît pas plutôt que de l'affronter de face, en combinant plusieurs de ces procédés lorsqu'un filtre bloque plus d'un motif.

## Outils

- **sqlmap** : les tamper scripts (par exemple `space2comment`, `charencode`) automatisent ces réécritures.
- **ghauri** : évasion de WAF intégrée pour les payloads SQLite.
- **Burp Repeater** : ajustez à la main les commentaires, `char()` et les littéraux blob jusqu'au contournement du filtre.

## Références

- Documentation SQLite : char, littéraux, commentaires, expressions
- OWASP Testing Guide: Testing for SQL Injection
