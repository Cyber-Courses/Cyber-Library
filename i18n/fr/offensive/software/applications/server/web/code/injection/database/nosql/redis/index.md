---
order: 4
title: "Injection Redis"
description: "Injection de commande via le protocole RESP et abus de commandes dangereuses dans un stockage clé-valeur Redis, du smuggling CRLF à la RCE par CONFIG SET et au Lua côté serveur."
keywords:
  - injection Redis
  - protocole RESP
  - injection CRLF
  - CONFIG SET
  - scripting Lua
---

# Redis

Redis est un stockage clé-valeur en mémoire parlé via **RESP**, un protocole de transport simple délimité par CRLF. L'injection contre Redis porte moins sur la syntaxe de requête que sur l'atteinte de la connexion : lorsqu'une entrée contrôlée par l'attaquant est embarquée dans une commande, ou lorsqu'un SSRF permet à un attaquant de parler directement à un port Redis, des séquences `\r\n` brutes font passer des commandes supplémentaires dans le flux.

Une fois que des commandes peuvent être smugglées, Redis offre des primitives à fort impact : `FLUSHALL` et `KEYS` pour l'accès de masse, `CONFIG SET` pour relocaliser le fichier de dump et écrire du contenu arbitraire sur disque, `SLAVEOF`/`REPLICAOF` pour tirer un jeu de données depuis un maître malveillant, `MODULE LOAD` là où les modules sont permis, et la famille `EVAL` pour le Lua côté serveur. Ce sous-arbre couvre le smuggling de commandes via RESP, la chaîne RCE d'écriture sur disque `CONFIG SET` et le scripting Lua via `EVAL`, `EVALSHA` et `SCRIPT LOAD`.

## Sous-thèmes

- **[Script Lua](lua-script/index.md)** : Redis exécute du Lua côté serveur via la famille EVAL, donnant à un attaquant qui contrôle un corps de script ou ses arguments un moteur de scripting à l'intérieur de la base de données.

## Pages

- **[Injection de commande](command.md)** : un CRLF non échappé dans l'entrée utilisateur, ou un SSRF vers un port Redis, permet à un attaquant d'ajouter des commandes RESP supplémentaires et d'atteindre FLUSHALL, CONFIG, SLAVEOF et MODULE LOAD.
- **[Abus de CONFIG SET](config-set-abuse.md)** : relocaliser le fichier de dump Redis avec CONFIG SET dir et dbfilename, puis SAVE, écrit un payload contrôlé sur disque : webshell, authorized_keys ou tâche cron.

## Outils

- **redis-cli** : le client Redis natif pour émettre et tester des commandes.
- **Gopherus** : générer des payloads RESP gopher:// pour les commandes smugglées par SSRF.
- **Burp Suite** : intercepter et altérer les requêtes web adossées à Redis.

## Références

- [Redis : command reference](https://redis.io/docs/latest/commands/)
- [PayloadsAllTheThings : Redis SSRF and command injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
