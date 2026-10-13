---
order: 3
title: "Dump en une seule requête (DIOS) pour l'injection MySQL"
description: "Construire un unique payload union MySQL qui concatène le schéma et les données dans une seule réponse avec des sous-requêtes imbriquées et GROUP_CONCAT."
keywords:
  - DIOS
  - dump in one shot
  - GROUP_CONCAT
  - extraction en une seule requête
  - extraction massive MySQL
---

# Dump en une seule requête

Le dump en une seule requête (DIOS) condense toute une énumération dans un unique payload union, de sorte qu'une seule requête retourne le schéma (et souvent les données) au lieu d'une requête distincte par table ou colonne. Il échange un payload long et fragile contre bien moins d'allers-retours, ce qui compte face aux limites de débit ou aux canaux aveugles lents.

Le cœur est une sous-requête qui parcourt `information_schema.columns` et utilise `GROUP_CONCAT` pour aplatir chaque `table.column` en une seule chaîne, déposée dans une position reflétée :

```sql
' UNION SELECT NULL,(SELECT GROUP_CONCAT(table_name,0x2e,column_name SEPARATOR 0x0a) FROM information_schema.columns WHERE table_schema=database()),NULL-- 
```

Les payloads DIOS plus élaborés ajoutent du formatage (sauts de ligne HTML, délimiteurs) afin que le dump soit lisible dans la page rendue, et certains imbriquent un second `GROUP_CONCAT` pour tirer les données réelles des lignes des tables découvertes dans la même requête.

La limite pratique est `group_concat_max_len` (1024 octets par défaut), qui tronque les gros dumps. Augmentez-la avec `SET SESSION group_concat_max_len=1000000` là où le compte le permet, ou repliez-vous sur une extraction paginée pour les tables qui débordent. DIOS est une commodité bâtie sur les mêmes primitives `information_schema` et `GROUP_CONCAT` que l'extraction union ordinaire, pas une vulnérabilité distincte.

## Outils

- **sqlmap** : extraction union automatisée qui remplace les payloads DIOS construits à la main.
- **Burp Repeater** : ajuster et rejouer manuellement le payload DIOS à requête unique.

## Références

- MySQL Reference Manual: GROUP_CONCAT, `group_concat_max_len`, `information_schema`
- OWASP Testing Guide: Testing for SQL Injection
