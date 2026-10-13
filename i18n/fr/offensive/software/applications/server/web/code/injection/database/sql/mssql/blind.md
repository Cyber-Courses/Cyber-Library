---
order: 1
title: "Injection SQL aveugle basée sur booléen dans MSSQL"
description: "Inférer les données SQL Server caractère par caractère à partir des différences de réponse vrai/faux à l'aide de SUBSTRING, ASCII et LEN."
keywords:
  - blind basé sur booléen
  - injection SQL aveugle
  - SUBSTRING ASCII
  - inférence MSSQL
  - recherche binaire
---

# Aveugle

Lorsque la réponse ne reflète ni les lignes ni le texte d'erreur, une condition dont la véracité dépend des données les divulgue quand même par le comportement. Si une condition vraie et une condition fausse s'affichent différemment, chaque comparaison injectée répond à une question oui/non.

SQL Server isole un caractère avec `SUBSTRING()` et le convertit avec `ASCII()` (ou `UNICODE()` pour l'Unicode) pour la comparaison :

```sql
' AND ASCII(SUBSTRING((SELECT TOP 1 name FROM sys.sql_logins),1,1))>77-- 
```

Une recherche binaire fixe chaque caractère en environ sept requêtes, puis la position avance. Trouvez d'abord la longueur avec `LEN()` :

```sql
' AND LEN((SELECT TOP 1 name FROM sys.sql_logins))=4-- 
```

Confirmez l'oracle avec une paire vrai/faux connue (`' AND 1=1--` contre `' AND 1=2--`). `TOP 1` sélectionne une seule ligne parce que SQL Server n'a pas de `LIMIT`, et `ORDER BY` avec `OFFSET ... FETCH` parcourt les lignes successives quand plus que la première est nécessaire.

Parce que cela génère de nombreuses requêtes quasi identiques, c'est presque toujours automatisé, mais la comparaison à requête unique est la primitive qui permet de s'adapter quand un outil cale, et elle bascule proprement vers l'inférence temporelle lorsqu'aucune différence booléenne n'est visible.

## Outils

- **sqlmap** : extraction aveugle automatisée basée sur booléen avec recherche binaire.
- **ghauri** : alternative rapide avec forte évasion de WAF.

## Références

- Microsoft SQL Server Documentation : SUBSTRING, ASCII, LEN, TOP
- PortSwigger Web Security Academy : Blind SQL injection
