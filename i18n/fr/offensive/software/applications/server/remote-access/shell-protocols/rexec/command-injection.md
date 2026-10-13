---
order: 1
title: "Injection de commande : abuser d'une construction de commande rexec non sécurisée"
description: "rexec exécute une chaîne de commande sur le serveur, et là où cette chaîne est construite à partir d'une entrée influencée par l'attaquant par un wrapper, un script ou une application qui appelle rexec de façon non sécurisée, les métacaractères shell injectent des commandes supplémentaires. Cela étend une unique commande prévue en exécution arbitraire sous le compte qui l'exécute."
keywords:
  - command injection
  - rexec
  - shell metacharacters
  - unsafe construction
  - rce
---

# Injection de commande

rexec exécute une chaîne de commande sur la cible, et l'injection de commande survient non pas dans le protocole lui-même, mais partout où quelque chose construit cette chaîne à partir d'une entrée influencée par l'attaquant sans assainissement. Les scripts d'administration, les front-ends web et les applications qui font appel à rexec (ou à la commande côté serveur) en concaténant l'entrée utilisateur sont la couche vulnérable : injecter des métacaractères shell (`;`, `|`, `` ` ``, `$()`) dans un paramètre qui aboutit dans la commande exécutée ajoute des commandes de l'attaquant, qui s'exécutent sous le compte qui les exécute. Cela transforme une opération unique prévue en exécution de commande arbitraire.

```bash
# where an app/script passes user input into a rexec'd command unsafely:
#   intended:  rexec ... host "report <USERINPUT>"
#   injected:  USERINPUT = "x; id; cat /etc/passwd"
rexec -l user -p pass <target> 'report x; id; cat /etc/passwd'
# or server-side, where rexecd runs a command that itself concatenates attacker data
#   supply metacharacters in the field that reaches the shell
```

## Notes d'exploitation

- La faille est dans l'appelant, pas dans rexecd : cherchez les scripts, tâches cron ou applications qui construisent une commande rexec (ou côté serveur) à partir de paramètres que vous contrôlez, et injectez des métacaractères shell dans ces paramètres.
- Les commandes injectées s'exécutent sous le compte, quel qu'il soit, qui réalise l'exécution, ce qui, pour l'outillage d'administration automatisé, est souvent privilégié.
- C'est la même classe d'injection que toute construction de commande shell non sécurisée ; rexec n'est que le sink. Vérifiez si un shell interprète la chaîne (les métacaractères n'aident que si un shell les analyse).
- Combinez avec la [capture en clair](cleartext-passwords.md) (pour obtenir l'identifiant utilisé par le wrapper) ou la [confiance](trusted-host-bypass.md) pour atteindre le chemin injectable.

## Références

- [OWASP: command injection](https://owasp.org/www-community/attacks/Command_Injection)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
