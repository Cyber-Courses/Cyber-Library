---
title: "SCRIPT LOAD : mise en cache d'un script pour obtenir son SHA1"
description: "SCRIPT LOAD compile et met en cache un script Lua sans l'exécuter et renvoie son SHA1, amorçant le cache pour un EVALSHA ultérieur dans un workflow de type load-then-exec."
keywords:
  - SCRIPT LOAD
  - EVALSHA
  - empreinte SHA1
  - amorçage du cache de scripts
  - load then exec
---

# SCRIPT LOAD

`SCRIPT LOAD <script>` compile un script Lua, le stocke dans le cache de scripts du serveur et renvoie son empreinte SHA1, le tout **sans l'exécuter**. C'est la moitié « préparation » du workflow de scripting : le script reste prêt dans le cache, indexé par `SHA1(script_body)`, en attente d'être déclenché par [EVALSHA](evalsha.md).

## Le workflow load-then-exec

Offensivement, `SCRIPT LOAD` est intéressant précisément parce qu'il sépare la préparation de l'exécution. Un attaquant qui atteint un point d'injection autorisant `SCRIPT LOAD` peut placer un script malveillant et paramétré dans le cache et noter l'empreinte renvoyée :

```
SCRIPT LOAD "return redis.call(unpack(ARGV))"
# -> "5f2e...c8"
```

Le corps chargé est un répartiteur de commandes générique : `unpack(ARGV)` déploie le nombre d'arguments fournis dans `redis.call`, de sorte qu'il invoque le nom de commande et les arguments passés au moment de l'exécution avec la bonne arité (une forme fixe `ARGV[1],ARGV[2],ARGV[3]` passerait un `nil` final et Redis rejette les arguments nil). Un seul script en cache pilote alors de nombreuses commandes **autorisées en script** via `EVALSHA` :

```
EVALSHA 5f2e...c8 0 KEYS *
EVALSHA 5f2e...c8 0 GET sessions:admin
EVALSHA 5f2e...c8 0 SET sessions:admin forged-value
```

Le répartiteur reste contraint par le bac à sable de scripting : les commandes `noscript` telles que `CONFIG`, `SLAVEOF`/`REPLICAOF` et `DEBUG` sont rejetées à l'intérieur du script, donc un chemin de configuration admin ou d'écriture sur disque (voir [abus de CONFIG SET](../config-set-abuse.md)) doit être piloté par des commandes directes, non via ce répartiteur chargé.

Comme l'empreinte est déterministe, l'attaquant peut la calculer hors ligne à partir du corps voulu et commencer à émettre des `EVALSHA` immédiatement après la réussite du chargement, sans lire la réponse du chargement. C'est utile sur des canaux aveugles ou unidirectionnels comme une connexion RESP acheminée par SSRF (voir [Injection de commandes](../command.md)), où la sortie des commandes peut ne pas revenir alors que le script préparé s'exécute tout de même.

## Amorçage pour une chaîne à points d'injection séparés

Le chargement et l'exécution n'ont pas besoin de partager une connexion ni un point d'injection. Lorsqu'une requête peut déclencher `SCRIPT LOAD` et qu'une autre requête peut déclencher `EVALSHA`, le cache est partagé à l'échelle du serveur, donc un script préparé via la première est exécutable via la seconde. Cela découple les deux moitiés de l'attaque et permet à une primitive d'écriture éphémère d'amorcer une charge utile qu'une primitive ultérieure, plus limitée, invoque.

## Inspection et persistance

`SCRIPT EXISTS <sha1> [sha1 ...]` indique si les empreintes données sont en cache (renvoyant `1` ou `0` pour chacune), utile pour confirmer qu'une préparation a abouti avant de la déclencher. Le cache est en mémoire et vidé par `SCRIPT FLUSH` ou un redémarrage du serveur, donc une empreinte préparée peut nécessiter un rechargement après l'un ou l'autre. Charger le même corps à nouveau est idempotent : cela renvoie le même SHA1 et laisse l'état du cache inchangé, donc le réamorçage est peu coûteux.

## Outils

- **redis-cli** : préparer un script avec SCRIPT LOAD et le confirmer avec SCRIPT EXISTS.
- **Gopherus** : acheminer le SCRIPT LOAD sur une connexion RESP via SSRF.
- **Burp Repeater** : délivrer la requête de chargement au point d'injection.

## Références

- [Redis : SCRIPT LOAD](https://redis.io/docs/latest/commands/script-load/)
- [Redis : SCRIPT EXISTS](https://redis.io/docs/latest/commands/script-exists/)
- [Redis : EVALSHA](https://redis.io/docs/latest/commands/evalsha/)
