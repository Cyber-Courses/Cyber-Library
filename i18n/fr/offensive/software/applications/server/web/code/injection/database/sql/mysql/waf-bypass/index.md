---
order: 11
title: "Contournement de WAF et de filtre pour l'injection MySQL"
description: "Atteindre les mots-clés, fonctions et tables de catalogue MySQL malgré des filtres par signature via les commentaires versionnés, l'encodage, les objets alternatifs et les astuces de jeu de caractères."
keywords:
  - contournement de WAF
  - évasion de filtre MySQL
  - commentaires versionnés
  - injection sur caractère large
  - alternative à information_schema
---

# Contournement de WAF

Les pare-feux applicatifs web et les filtres d'entrée bloquent l'injection en faisant correspondre des signatures : des mots-clés comme `UNION SELECT`, des noms de fonctions et la chaîne `information_schema`. Le dialecte de MySQL offre suffisamment d'équivalents pour que la plupart des filtres à signature unique puissent être contournés, ce qui explique pourquoi les listes noires de mots-clés sont une défense faible.

Les grandes familles sont : les astuces de commentaires qui découpent ou exécutent conditionnellement les mots-clés, les encodages qui retirent guillemets et espaces du payload, les objets alternatifs qui atteignent les mêmes données sous un autre nom, et les astuces de jeu de caractères qui neutralisent l'échappement. On les combine au besoin, car un filtre bloque généralement plusieurs motifs à la fois.

## Pages

- **[Commentaires versionnés](versioned-comments.md)** : `/*! */` et `/**/` pour découper et exécuter conditionnellement des mots-clés.
- **[Injection sur caractère large](wide-byte-gbk.md)** : séquences multioctets GBK qui absorbent les antislashs d'échappement.
- **[Catalogue et fonctions alternatifs](alternative-catalog.md)** : atteindre le schéma et la version sans les noms bloqués.

## Outils

- **sqlmap** : nombreux scripts tamper pour l'évasion de filtre et de WAF.
- **Burp Repeater** : fabriquez et itérez manuellement des payloads de contournement.

## Références

- MySQL Reference Manual : comment syntax, character sets
- OWASP Testing Guide : Testing for SQL Injection
