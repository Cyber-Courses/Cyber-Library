---
order: 1
title: "Injection SQL aveugle basée sur les booléens dans Oracle"
description: "Déduire des données Oracle caractère par caractère à partir des différences de réponse vrai/faux via SUBSTR, ASCII et LENGTH sur DUAL."
keywords:
  - aveugle basée sur les booléens
  - injection SQL aveugle
  - SUBSTR ASCII
  - inférence Oracle
  - DUAL
---

# Aveugle

Lorsque la réponse ne reflète ni les lignes ni le texte d'erreur, une condition dont la véracité dépend des données les divulgue tout de même via le comportement. Si une condition vraie et une condition fausse s'affichent différemment, chaque comparaison injectée répond à une question oui/non.

Oracle isole un caractère avec `SUBSTR()` et le convertit avec `ASCII()` pour la comparaison. Les sous-requêtes sélectionnent `FROM DUAL` ou une table cible :

```sql
' AND ASCII(SUBSTR((SELECT user FROM dual),1,1))>77-- 
```

Une recherche binaire fixe chaque caractère en environ sept requêtes, puis la position avance. Trouvez d'abord la longueur avec `LENGTH()` :

```sql
' AND LENGTH((SELECT user FROM dual))=5-- 
```

Confirmez l'oracle avec une paire vrai/faux connue (`' AND 1=1--` contre `' AND 1=2--`). Lire depuis une source à plusieurs lignes nécessite un sélecteur d'une seule ligne, puisque Oracle n'a pas de `LIMIT` :

```sql
' AND ASCII(SUBSTR((SELECT password FROM sys.user$ WHERE ROWNUM=1),1,1))>64-- 
```

Parce que cela génère de nombreuses requêtes quasi identiques, c'est presque toujours automatisé, mais la comparaison à requête unique est la primitive qui vous permet de vous adapter quand un outil cale, et elle se replie sur l'inférence temporelle quand aucune différence booléenne n'est visible.

## Outils

- **sqlmap** : extraction booléenne automatisée (`--technique=B`).
- **ghauri** : inférence booléenne rapide avec une forte évasion de WAF.
- **Burp Intruder** : scripte à la main les requêtes de comparaison par caractère.

## Références

- Oracle Database SQL Language Reference : SUBSTR, ASCII, LENGTH, ROWNUM
- PortSwigger Web Security Academy : Blind SQL injection
