---
title: "Extraction de caractères booléenne dans l'injection aveugle PostgreSQL"
description: "L'oracle de caractères substring et ascii pour l'injection SQL aveugle PostgreSQL, extrayant les valeurs par une recherche binaire sur les points de code."
keywords:
  - substring
  - ascii
  - recherche binaire
  - extraction aveugle
  - oracle de caractères
---

# Extraction booléenne

L'oracle de caractères lit une valeur une position à la fois. `substring(value FROM pos FOR 1)` (ou `substring(value, pos, 1)`) renvoie le caractère à `pos`, et `ascii(...)` donne son point de code pour une comparaison que la réponse révèle comme vraie ou fausse.

Extrayez une valeur par une recherche binaire sur chaque position. Pour le premier caractère de l'empreinte du mot de passe du premier rôle :

```sql
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))>64-- 
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))>96-- 
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))=116-- 
```

Chaque test `>` divise par deux la plage de candidats, de sorte qu'un caractère imprimable se résout en environ sept requêtes. Avancez la position `substring` pour parcourir la chaîne et arrêtez-vous à la longueur trouvée plus tôt.

Quand les opérateurs de comparaison sont filtrés, `substring(...) = 't'` teste l'égalité directement, et la forme SQL-standard `substring(value FROM pos FOR 1)` évite la virgule que certains filtres bloquent. Face à un rôle non privilégié qui ne peut pas lire `pg_shadow`, pointez la sous-requête vers une table applicative à la place. Comme toute extraction aveugle, celle-ci est fortement automatisée, mais la comparaison par requête est la primitive fiable sous-jacente à l'outillage.

## Outils

- **sqlmap** : extraction aveugle booléenne automatisée avec recherche binaire.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- PostgreSQL Documentation: `substring`, `ascii`
- PortSwigger Web Security Academy: Blind SQL injection
