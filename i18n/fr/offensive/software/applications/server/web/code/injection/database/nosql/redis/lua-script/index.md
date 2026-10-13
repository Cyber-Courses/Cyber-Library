---
order: 3
title: "Injection de script Lua"
description: "Redis exécute du Lua côté serveur via la famille EVAL, donnant à un attaquant qui contrôle un corps de script ou ses arguments un moteur de scripting à l'intérieur de la base de données."
keywords:
  - Lua Redis
  - EVAL
  - EVALSHA
  - SCRIPT LOAD
  - scripting côté serveur
---

# Script Lua

Redis embarque un interpréteur Lua et exécute des scripts côté serveur via la famille de commandes `EVAL`. Un script peut rappeler Redis avec `redis.call()`, boucler sur les clés, transformer des valeurs et renvoyer des résultats arbitraires, le tout atomiquement sur le serveur. Lorsqu'un attaquant contrôle un corps de script, ou les arguments passés à l'un d'eux, ce moteur de scripting devient une surface d'injection à l'intérieur de la base de données.

Les scripts atteignent le serveur de trois façons : `EVAL` exécute un script fourni inline, `SCRIPT LOAD` met un script en cache et renvoie son digest SHA1, et `EVALSHA` exécute un script précédemment mis en cache par ce digest. Les trois sont liés : un script chargé une fois avec `SCRIPT LOAD` peut être ré-exécuté à bas coût par SHA1, et un attaquant qui peut charger un script peut l'invoquer plus tard. Ce sous-arbre couvre chaque commande en tant que vecteur d'injection.

## Pages

- **[EVAL](eval.md)** : EVAL exécute un script Lua à l'intérieur de Redis ; l'injection dans le corps du script ou ses arguments permet à un attaquant de piloter redis.call() et d'atteindre chaque clé côté serveur.
- **[EVALSHA](evalsha.md)** : EVALSHA exécute un script Lua déjà en cache dans Redis par son digest SHA1, permettant à un attaquant de ré-invoquer un script qu'il a chargé ou un script déjà présent dans le cache...
- **[SCRIPT LOAD](script-load.md)** : SCRIPT LOAD compile et met en cache un script Lua sans l'exécuter et renvoie son SHA1, semant le cache pour un EVALSHA ultérieur dans un workflow charger-puis-exécuter.

## Outils

- **redis-cli** : émettre EVAL, SCRIPT LOAD et EVALSHA contre le serveur.
- **Gopherus** : faire passer des commandes de scripting via une connexion RESP SSRF.
- **Burp Suite** : intercepter et altérer les requêtes web adossées à Redis.

## Références

- [Redis : scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
