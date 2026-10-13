---
order: 1
title: "Injection SQL aveugle basée sur booléen dans IBM Db2"
description: "Inférer les données IBM Db2 caractère par caractère à partir des différences de réponse vrai/faux à l'aide de SUBSTR et ASCII sur SYSIBM.SYSDUMMY1."
keywords:
  - blind basé sur booléen
  - injection SQL aveugle
  - SUBSTR ASCII
  - inférence Db2
  - SYSIBM.SYSDUMMY1
---

# Aveugle

Parce que le canal d'erreur de Db2 est étroit, l'inférence booléenne est le moyen fiable de lire les données quand les lignes ne sont pas reflétées. Si une condition vraie et une condition fausse s'affichent différemment, chaque comparaison injectée répond à une question oui/non.

Db2 isole un caractère avec `SUBSTR()` et le convertit avec `ASCII()` pour la comparaison. Les sous-requêtes qui retournent un scalaire sélectionnent `FROM SYSIBM.SYSDUMMY1` ou une table cible :

```sql
' AND ASCII(SUBSTR((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1),1,1))>77-- 
```

Une recherche binaire fixe chaque caractère en environ sept requêtes, puis la position avance. Trouvez d'abord la longueur avec `LENGTH()` :

```sql
' AND LENGTH((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1))=8-- 
```

Confirmez l'oracle avec une paire vrai/faux connue (`' AND 1=1--` contre `' AND 1=2--`). Lire depuis une source multi-lignes nécessite un sélecteur de ligne unique, puisque Db2 n'a pas de `LIMIT` :

```sql
' AND ASCII(SUBSTR((SELECT password FROM users ORDER BY id FETCH FIRST 1 ROWS ONLY),1,1))>64-- 
```

Parce que cela génère de nombreuses requêtes quasi identiques, c'est presque toujours automatisé, mais la comparaison à requête unique est la primitive qui permet de s'adapter quand un outil cale. C'est le canal principal contre Db2 vu la surface d'erreur limitée, avec l'inférence temporelle comme solution de repli lorsqu'aucune différence booléenne n'est visible.

## Outils

- **sqlmap** : extraction automatisée basée sur booléen (`--technique=B`).
- **ghauri** : inférence booléenne rapide avec forte évasion de WAF.
- **Burp Intruder** : scripte à la main les requêtes de comparaison par caractère.

## Références

- IBM Db2 SQL Reference : SUBSTR, ASCII, LENGTH, FETCH FIRST
- PortSwigger Web Security Academy : Blind SQL injection
