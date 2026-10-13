---
title: "Injection temporelle MySQL avec BENCHMARK"
description: "Utiliser BENCHMARK pour créer des délais liés au CPU dans une injection SQL aveugle MySQL quand SLEEP est filtré, et pourquoi c'est moins fiable que SLEEP."
keywords:
  - injection BENCHMARK
  - délai CPU
  - SLEEP filtré
  - MySQL basée sur le temps
---

# BENCHMARK

`BENCHMARK(count, expr)` évalue `expr` `count` fois et renvoie 0. Il ne produit aucune valeur utile, mais exécuter une expression coûteuse un nombre suffisant de fois prend un temps mesurable à l'horloge, ce qui fournit un oracle de délai lorsque `SLEEP` est bloqué par un filtre.

Rendez-le conditionnel avec `IF()` afin que le CPU ne soit sollicité que sur un test vrai. Associez-le à une expression délibérément coûteuse comme un hash pour que chaque itération effectue un vrai travail :

```sql
' AND IF(ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>77,BENCHMARK(5000000,SHA1(RAND())),0)-- 
```

Contrairement à `SLEEP`, le délai n'est pas un nombre fixe de secondes ; il dépend de la vitesse du serveur et du nombre d'itérations, de sorte que le compteur doit être ajusté jusqu'à ce qu'un test vrai ajoute un délai net et reproductible (souvent quelques millions d'itérations pour une à deux secondes). Cela rend `BENCHMARK` plus bruyant et plus lent à calibrer que `SLEEP`, c'est donc un recours plutôt qu'un premier choix.

Comme il occupe un cœur CPU pour chaque sonde, des nombres d'itérations élevés sur de nombreux caractères imposent une charge réelle au serveur. Gardez le compteur juste assez élevé pour obtenir un délai distinguable.

## Outils

- **sqlmap** : extraction temporelle automatisée, repliant sur BENCHMARK quand SLEEP est filtré.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- MySQL Reference Manual : BENCHMARK, SHA1, IF
- PortSwigger Web Security Academy : SQL injection cheat sheet
