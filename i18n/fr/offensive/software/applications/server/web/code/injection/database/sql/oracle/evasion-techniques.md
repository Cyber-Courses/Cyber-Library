---
order: 4
title: "Évasion de filtre et de WAF pour l'injection Oracle"
description: "Évader les filtres dans l'injection Oracle avec les commentaires en ligne, la construction de chaînes par CHR(), la concaténation et la manipulation de casse."
keywords:
  - contournement de WAF
  - fonction CHR
  - commentaires en ligne
  - concaténation
  - évasion de filtre Oracle
---

# Techniques d'évasion

La syntaxe d'Oracle offre plusieurs moyens de passer devant les filtres par signature qui bloquent les guillemets ou les mots-clés.

Les chaînes sont construites sans guillemets à partir de codes de caractères avec `CHR()` joints par `||`, ce qui défait les filtres de guillemets et les signatures de chaînes entre guillemets :

```sql
-- 'SCOTT' without quotes
CHR(83)||CHR(67)||CHR(79)||CHR(84)||CHR(84)
```

Les commentaires en ligne `/**/` remplacent les espaces, brisant les signatures qui font correspondre des jetons séparés par des espaces, et la concaténation découpe un mot-clé qu'un filtre cherche comme un unique littéral de chaîne :

```sql
' UNION/**/SELECT/**/banner/**/FROM/**/v$version-- 
```

La casse est libre de varier puisque les mots-clés Oracle sont insensibles à la casse (`UnIoN SeLeCt`), ce qui contourne les listes noires naïves sensibles à la casse. Les nombres peuvent être écrits sous des formes alternatives, et les valeurs peuvent être reconstruites à partir de conversions `CHR`, `CONCAT` ou `UTL_RAW`/`TO_CHAR` pour éviter les sous-chaînes filtrées.

`CASE WHEN`/`DECODE` restructurent une condition pour qu'elle ne corresponde plus à un motif tout en conservant la même logique, ce qui aide quand un opérateur de comparaison ou un mot-clé spécifique est filtré. Comme avec les autres moteurs, l'objectif est d'exprimer la même requête via des synonymes et des encodages que le filtre ne reconnaît pas, plutôt que de défaire le filtre de front, et ces techniques se combinent librement car un filtre bloque généralement plusieurs motifs à la fois.

## Outils

- **sqlmap** : des scripts tamper (par exemple `space2comment`, `charencode`) automatisent ces réécritures.
- **ghauri** : évasion de WAF intégrée pour les payloads Oracle.
- **Burp Repeater** : ajustez à la main les commentaires, `CHR()` et la casse jusqu'à contourner le filtre.

## Références

- Oracle Database SQL Language Reference : CHR, comments, expressions
- OWASP Testing Guide : Testing for SQL Injection
