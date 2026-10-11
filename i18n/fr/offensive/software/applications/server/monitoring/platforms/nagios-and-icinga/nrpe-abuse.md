---
title: "Abus de NRPE : l'exécuteur de plugins distant de Nagios"
order: 2
description: "NRPE, sur le port TCP 5666, exécute des commandes de vérification prédéfinies sur un hôte supervisé pour le compte du serveur Nagios. Lorsque l'agent autorise les arguments de commande (dont_blame_nrpe) et qu'une commande les transmet à un shell, un attaquant qui atteint le port 5666 injecte des arguments et des métacaractères du shell pour exécuter des commandes arbitraires sur l'hôte, transformant un agent de supervision en exécution de code à distance."
keywords:
  - nrpe
  - 5666
  - dont_blame_nrpe
  - argument injection
  - command execution
---

# Abus de NRPE

NRPE (Nagios Remote Plugin Executor) écoute sur le port TCP 5666 des hôtes supervisés et exécute des commandes de vérification prédéfinies pour le compte du serveur Nagios, puis renvoie le résultat. Par défaut, il n'exécute que les commandes définies dans sa configuration et n'accepte pas d'arguments, mais deux assouplissements courants l'ouvrent. Si `dont_blame_nrpe=1` (autoriser les arguments) est activé, le serveur peut transmettre des arguments aux définitions de commandes qui utilisent `$ARG1$`, etc., et si une définition de commande transmet ces arguments à un shell ou à un plugin qui les interprète, un attaquant capable d'atteindre le port 5666 injecte des métacaractères du shell à travers les arguments pour exécuter des commandes arbitraires sur l'hôte. Même sans passage d'arguments, une commande mal définie ou un plugin qui invoque un shell sur une entrée influencée par l'attaquant est exploitable. Le résultat est l'exécution de commandes sur l'hôte supervisé sous l'identité de l'utilisateur NRPE.

```bash
# confirm the agent and list reachable behaviour
check_nrpe -H <target>                            # version; no command = agent present
check_nrpe -H <target> -c check_users             # run a defined command
# argument injection where dont_blame_nrpe=1 and a command passes args to a shell:
check_nrpe -H <target> -c <cmd_using_ARG> -a 'arg; id'         # ; command injection
check_nrpe -H <target> -c check_command -a '$(id)'            # if args reach a shell
# classic metachars that break out where the plugin/command invokes a shell: ; | ` $() &&
```

## Notes d'exploitation

- La condition préalable est le passage d'arguments (`dont_blame_nrpe=1`) plus une commande dont la définition transmet l'argument à un shell ou à un plugin interprétatif ; testez les commandes définies avec des métacaractères injectés dans les arguments `-a`.
- Atteindre le port 5666 est souvent possible depuis l'intérieur du réseau (les agents sont filtrés par pare-feu vers le serveur, mais les réseaux à plat et les mauvaises configurations les exposent), et un serveur Nagios compromis peut atteindre chaque agent ; c'est un fort point de pivot vers tous les hôtes supervisés.
- L'exécution a lieu sous l'identité de l'utilisateur de service NRPE sur l'hôte supervisé ; servez-vous-en pour lire des fichiers, récupérer des identifiants et vous déplacer latéralement.
- Il s'agit de la surface de commande côté agent ; l'injection dans les définitions de commandes côté serveur relève de l'[injection de commandes et de plugins](command-and-plugin-injection.md).

## Outils

- [check_nrpe](https://github.com/NagiosEnterprises/nrpe)

## Références

- [Sécurité de NRPE (dont_blame_nrpe)](https://github.com/NagiosEnterprises/nrpe/blob/master/SECURITY.md)
- [HackTricks : NRPE](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
