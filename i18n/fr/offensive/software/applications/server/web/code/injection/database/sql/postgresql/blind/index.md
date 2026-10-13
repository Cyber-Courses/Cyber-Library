---
order: 1
title: "Injection SQL aveugle booléenne dans PostgreSQL"
description: "Déduire des données PostgreSQL un caractère à la fois à partir de différences vrai/faux dans la réponse quand aucune ligne ni erreur n'est reflétée."
keywords:
  - boolean based blind
  - injection SQL aveugle
  - inférence PostgreSQL
  - substring ascii
  - recherche binaire
---

# Aveugle booléenne

Quand l'application ne reflète ni les lignes ni le texte d'erreur, une condition dont la véracité dépend des données cibles les fuit tout de même via la réponse. Si une condition vraie et une condition fausse s'affichent différemment, chaque comparaison injectée répond à une question oui/non, et un nombre suffisant de questions reconstitue n'importe quelle valeur.

PostgreSQL isole un caractère avec `substring()` et le convertit avec `ascii()` pour la comparaison :

```sql
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))>77-- 
```

Une recherche binaire fixe chaque caractère en environ sept requêtes, puis la position avance. Confirmez d'abord l'oracle avec une paire vrai/faux connue :

```sql
' AND 1=1-- 
' AND 1=2-- 
```

Déterminez la longueur avec `length()` afin de savoir quand vous arrêter :

```sql
' AND length((SELECT passwd FROM pg_shadow LIMIT 1))=32-- 
```

Le même oracle lit `current_user`, `current_database()` et les données de tables en changeant la sous-requête interne. L'extraction aveugle est lente et presque toujours automatisée, mais la comparaison par requête unique est ce qui vous permet de vous adapter quand un outil bloque.

## Pages

- **[Extraction booléenne](boolean-based.md)** : l'oracle de caractères `substring`/`ascii` avec recherche binaire.

## Outils

- **sqlmap** : extraction aveugle booléenne automatisée.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- PostgreSQL Documentation: `substring`, `ascii`, `length`
- PortSwigger Web Security Academy: Blind SQL injection
