---
title: "EVALSHA : exécuter des scripts en cache par SHA1"
description: "EVALSHA exécute un script Lua déjà en cache dans Redis par le digest SHA1 de son corps, permettant à un attaquant de ré-invoquer un script qu'il a chargé ou un script déjà présent dans le cache."
keywords:
  - EVALSHA
  - SCRIPT LOAD
  - digest SHA1
  - cache de scripts Redis
  - réutilisation de script en cache
---

# EVALSHA

`EVALSHA <sha1> <numkeys> [key ...] [arg ...]` exécute un script Lua déjà en cache dans le cache de scripts du serveur, identifié par le digest SHA1 de son corps. C'est le jumeau économe en bande passante d'`EVAL` : le client envoie un hash de 40 caractères au lieu du script complet, et Redis exécute la copie en cache avec une sémantique identique.

## Relation avec SCRIPT LOAD et EVAL

Un script entre dans le cache de deux façons. `EVAL` met son script en cache comme effet de bord de son exécution, et `SCRIPT LOAD <script>` met un script en cache **sans** l'exécuter et renvoie son SHA1 (voir [SCRIPT LOAD](script-load.md)). Dans les deux cas, le digest est déterministe : c'est `SHA1(script_body)`, de sorte que le même script a toujours la même clé. Après un chargement, le script peut être déclenché de façon répétée par digest :

```
SCRIPT LOAD "return redis.call('KEYS','*')"
# -> "a9b1c3...e7"
EVALSHA a9b1c3...e7 0
```

Comme le digest ne dépend que du corps, un attaquant peut le précalculer hors ligne. Si un script au corps connu est déjà en cache, `EVALSHA` l'exécute sans que l'attaquant n'ait jamais fourni le texte.

## Réutilisation de scripts chargés par l'attaquant

Le flux offensif courant est charger-puis-exécuter à travers deux sinks atteignables. Lorsqu'un point d'injection autorise `SCRIPT LOAD` (ou un `EVAL` antérieur) et qu'un autre autorise `EVALSHA`, l'attaquant met en scène un script malveillant une fois et le déclenche plus tard :

```
SCRIPT LOAD "return redis.call('SET', KEYS[1], ARGV[1])"
# cache now holds a key-writing script under its SHA1
EVALSHA <sha1> 1 sessions:admin "forged-value"
```

Le script mis en scène est paramétré via `KEYS`/`ARGV`, de sorte qu'un seul corps en cache pilote plusieurs actions en variant les arguments à chaque `EVALSHA`.

Notez le plafond : `EVAL`/`EVALSHA` ne peuvent invoquer que des commandes **autorisées par script**. `CONFIG` (SET/GET), `SLAVEOF`/`REPLICAOF` et `DEBUG` portent le flag `noscript` et sont rejetées à l'intérieur d'un script Lua avec « This Redis command is not allowed from script », de sorte que la séquence RCE d'écriture sur disque de [Abus de CONFIG SET](../config-set-abuse.md) ne peut pas être mise en scène via Lua et doit être émise sous forme de commandes **directes**. Ce qu'`EVALSHA` met en scène, c'est de la manipulation du plan de données (`GET`/`SET`/`KEYS`/`MGET`/`SCAN`) sur un sink aveugle ou scindé.

## NOSCRIPT et re-semis

Si le digest n'est pas en cache, Redis répond par une erreur `NOSCRIPT`. Cela signale que le script doit d'abord être semé, avec `SCRIPT LOAD` ou un `EVAL` complet, avant qu'`EVALSHA` ne l'exécute. Un flush de cache (`SCRIPT FLUSH`) ou un redémarrage de serveur efface tous les scripts en cache, de sorte qu'une chaîne `EVALSHA` mise en scène peut nécessiter un re-semis après l'un ou l'autre de ces événements. Lorsque seul `EVALSHA` est atteignable et que le corps est inconnu, énumérer des digests utiles est impraticable ; la voie pratique consiste à obtenir d'abord un sink `SCRIPT LOAD` ou `EVAL` pour contrôler le corps.

## Outils

- **redis-cli** : SCRIPT LOAD un corps et le déclencher par digest avec EVALSHA.
- **Burp Repeater** : déclencher l'exécution du script en cache via le sink.

## Références

- [Redis : EVALSHA](https://redis.io/docs/latest/commands/evalsha/)
- [Redis : SCRIPT LOAD](https://redis.io/docs/latest/commands/script-load/)
- [Redis : scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)
