---
title: "Substitution de commandes"
description: "Les backticks et $(...) exécutent une commande imbriquée et en insèrent la sortie dans la ligne de commande courante, même au milieu d'un argument."
keywords:
  - substitution de commandes
  - sous-shell
  - backticks
  - dollar parenthèses
  - commande imbriquée
---

# Substitution

La substitution de commandes, `` `cmd` `` et `$(cmd)`, exécute une commande imbriquée et en insère la sortie dans la ligne de commande environnante. Parce qu'elle est évaluée même lorsque le point d'injection se trouve au milieu d'un argument, elle est à la fois une primitive d'exécution principale et l'un des oracles de confirmation les plus fiables pour l'injection de commandes.

## Pages

- **[Backticks and $(...)](inside-command.md)** : utiliser $(...) et la substitution de commandes par backticks pour exécuter une commande imbriquée et en insérer la sortie dans la ligne de commande courante, fonctionnant même lorsque le point d'injection...

## Outils

- **[commix](https://github.com/commixproject/commix)** : exploitation automatisée des injections de commandes.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer les payloads de substitution.

## Références

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
