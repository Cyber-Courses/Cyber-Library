---
order: 1
title: "Injection de commandes OS"
description: "Le code applicatif construit une commande système à partir d'une entrée non fiable, ce qui permet à un attaquant de changer le programme exécuté ou ses arguments, généralement une exécution de code à distance."
keywords:
  - injection de commandes OS
  - injection de commandes
  - RCE
  - injection shell
  - injection d'arguments
---

# Commande

L'injection de commandes OS se produit lorsque le code applicatif construit une commande système à partir d'une entrée non fiable et la transmet au système d'exploitation d'une manière qui permet à l'attaquant de changer **le programme exécuté**, ou **ses arguments**. Le gain habituel est une exécution de code à distance dans le contexte du compte de service.

Le sous-arbre sépare les mécanismes de tout ce qui repose sur eux : l'enchaînement avec des séparateurs shell, la **substitution** de commandes et l'injection d'**arguments** (argv/flags) sans shell sont les voies d'entrée ; l'**exfiltration** couvre la récupération de données en aveugle et hors bande ; et le **contournement de filtres** couvre l'évasion des listes de blocage et des WAF.

## Sous-thèmes

- **[Argument](argument/index.md)** : une valeur utilisateur arrive dans argv et le binaire appelé la réinterprète comme une option plutôt que comme une donnée, sans shell, sans nouveau processus, et pourtant lecture/écriture de fichier et exécution de code.
- **[Chaining](chaining/index.md)** : les opérateurs de contrôle du shell qui permettent à une valeur injectée de terminer la commande prévue et d'en démarrer une autre : ; && || et &.
- **[Exfiltration](exfiltration/index.md)** : récupérer la sortie lorsqu'aucune n'est renvoyée : oracles temporels et canaux DNS/HTTP hors bande.
- **[Filter bypass](filter-bypass/index.md)** : les listes de blocage et les WAF sont contournés parce que le shell normalise un payload après que le filtre l'a inspecté, regroupés par ce que chaque technique déjoue.
- **[Substitution](substitution/index.md)** : les backticks et $(...) exécutent une commande imbriquée et en insèrent la sortie dans la ligne de commande courante, même au milieu d'un argument.

## Outils

- **[commix](https://github.com/commixproject/commix)** : détection et exploitation automatisées des injections de commandes.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour injecter et confirmer les payloads.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** : capture de callbacks hors bande pour les cas en aveugle.

## Références

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
