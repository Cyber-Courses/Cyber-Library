---
order: 1
title: "Contournement par absence d'espace et espaces"
description: "Déjouer les filtres basés sur l'espace et la ligne avec ${IFS}, l'expansion d'accolades et les continuations par saut de ligne ou antislash-saut de ligne."
keywords:
  - sans espace
  - IFS
  - no space
  - injection de saut de ligne
  - continuation de ligne
---

# Espaces

Techniques qui déjouent les filtres ciblant le caractère espace ou la structure de ligne : `${IFS}`, l'expansion d'accolades et la redirection d'entrée séparent les arguments sans espace littéral. Un saut de ligne non échappé termine une commande à l'intérieur d'un shell lancé, tandis qu'une continuation antislash-saut de ligne est retirée et les deux lignes physiques sont jointes avant la tokenisation, utile pour dissimuler un token à travers des lignes plutôt que pour terminer la commande.

## Pages

- **[Continuation antislash + saut de ligne](with-backslash-newline.md)** : Utiliser un antislash suivi d'un saut de ligne pour briser un mot-clé ou une commande à travers des lignes physiques pour qu'une blocklist littérale le manque tandis que le shell rejoint et exécut...
- **[Saut de ligne comme terminateur de commande](with-line-return.md)** : Injecter un saut de ligne brut ou encodé en URL (%0a, \\n) pour terminer la commande voulue et en démarrer une nouvelle à l'intérieur de sh -c, contournant les filtres qui ne bloquent que ; | &.
- **[Payloads sans espace](without-space.md)** : Produire whoami/id quand le caractère espace est filtré, en utilisant ${IFS}, $IFS$9, l'expansion d'accolades, la redirection d'entrée et des substituts de tabulation pour séparer une command...

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads sans espace.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements sans espace.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
