---
title: "Contournement de filtre d'injection de commande"
description: "Les blocklists et les WAF sont contournés parce que le shell normalise un payload après que le filtre l'a inspecté, regroupés selon ce que chaque technique déjoue."
keywords:
  - contournement de filtre
  - contournement de WAF
  - évasion d'injection de commande
  - obfuscation
  - contournement de blocklist
---

# Contournement de filtre

Les filtres de type blocklist et WAF invitent aux contournements parce que le shell normalise un payload **après** que le filtre l'a inspecté. Les techniques se regroupent selon ce qu'elles déjouent : produire une séparation d'arguments sans espaces (**espaces**), fragmenter des mots-clés bloqués pour qu'une correspondance littérale échoue (**fragmentation de mots-clés**), reconstruire des commandes et des chemins par expansion du shell (**expansion et globbing**), et dissimuler le payload par encodage ou variation de casse (**encodage et obfuscation**).

## Sous-thèmes

- **[Encodage et obfuscation](encoding-and-obfuscation/index.md)** : Dissimuler le payload par décodage hex/ANSI-C à l'exécution et variation de casse contre les blocklists sensibles à la casse.
- **[Expansion et globbing](expansion-and-globbing/index.md)** : Reconstruire des commandes et des chemins par expansion d'accolades, globbing par jokers, expansion du tilde et expansion de variables.
- **[Fragmentation de mots-clés](keyword-splitting/index.md)** : Fragmenter un mot-clé bloqué en morceaux que le shell rassemble avant l'exécution, guillemets, antislashs, substitutions vides, paramètres positionnels.
- **[Espaces](whitespace/index.md)** : Déjouer les filtres basés sur l'espace et la ligne avec ${IFS}, l'expansion d'accolades et les continuations par saut de ligne ou antislash-saut de ligne.

## Outils

- **[commix](https://github.com/commixproject/commix)** : les modules tamper automatisent les contournements de blocklist et de WAF.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour élaborer des payloads d'évasion.

## Références

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
