---
title: "Injection temporelle MySQL avec SLEEP"
description: "Payloads SLEEP conditionnels pour l'injection SQL aveugle MySQL, utilisant IF et des sous-requêtes corrélées afin que le délai ne se déclenche que lorsqu'un test est vrai."
keywords:
  - injection SLEEP
  - délai conditionnel
  - IF SLEEP
  - extraction temporelle MySQL
---

# SLEEP

`SLEEP(n)` met en pause pendant `n` secondes. Rendu conditionnel, il rapporte un bit : la réponse est lente quand le test est vrai et rapide quand il est faux.

La forme en ligne fiable enveloppe le délai dans `IF()`, qui est valide partout où une expression est autorisée et n'a pas besoin de `FROM` :

```sql
' AND IF(ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>77,SLEEP(5),0)-- 
```

Une réponse de cinq secondes signifie que le point de code du caractère est supérieur à 77 ; une réponse immédiate signifie qu'il ne l'est pas. Effectuez une recherche binaire sur chaque position exactement comme dans l'extraction booléenne, en lisant la latence au lieu du contenu de la page.

Lorsque l'injection se situe là où une sous-requête complète tient, un `SELECT` corrélé sur une table réelle avec le délai dans son `WHERE` est tout aussi valide, car le `FROM` fournit le contexte de ligne dont `SLEEP` a besoin :

```sql
' AND (SELECT 1 FROM users WHERE id=1 AND ASCII(SUBSTRING(password,1,1))>77 AND SLEEP(5))-- 
```

Évitez la forme erronée courante `AND (SELECT SLEEP(5) WHERE <test>)` : MySQL n'autorise un `SELECT` sans `FROM` que lorsqu'il renvoie des constantes, et ajouter un `WHERE` sans `FROM` est une erreur de syntaxe (contrairement à PostgreSQL et SQLite, qui l'acceptent), de sorte que cela lève l'erreur 1064 au lieu de temporiser. Gardez la requête interne à une seule ligne afin que `SLEEP` soit évalué de façon prévisible, et répétez une fois toute réponse positive pour écarter un réseau lent.

## Outils

- **sqlmap** : extraction temporelle automatisée utilisant un SLEEP conditionnel.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- MySQL Reference Manual : SLEEP, IF, subqueries
- PortSwigger Web Security Academy : Blind SQL injection
