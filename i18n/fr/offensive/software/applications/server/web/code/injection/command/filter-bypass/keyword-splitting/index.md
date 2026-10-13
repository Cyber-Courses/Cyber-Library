---
order: 2
title: "Contournement par fragmentation de mots-clés"
description: "Fragmenter un mot-clé bloqué en morceaux que le shell rassemble avant l'exécution, guillemets, antislashs, substitutions vides, paramètres positionnels."
keywords:
  - fragmentation de mots-clés
  - fragmentation par guillemets
  - contournement de blocklist
  - obfuscation
  - injection de commande
---

# Fragmentation de mots-clés

Fragmenter un mot-clé bloqué en morceaux que le shell supprime avant l'exécution. Les guillemets (`w"h"o"a"m"i`), les antislashs, les backticks vides ou `$()`, et les astuces de paramètres positionnels insèrent tous des caractères que le shell supprime durant l'expansion, de sorte qu'une blocklist à chaîne littérale ne voit jamais le mot interdit.

## Pages

- **[Insertion d'antislash et de slash](with-backslash-and-slash.md)** : Fragmenter un mot-clé bloqué avec des échappements par antislash, w\\ho\\am\\i, et insérer des slashs redondants dans les chemins, /bin/c\\at, //bin//cat, pour que le shell normalise...
- **[Backticks vides](with-backticks.md)** : Fragmenter un mot-clé de commande filtré avec une substitution de backticks vide, wh``oami, pour que le shell supprime la commande vide et réassemble le mot-clé, déjou...
- **[Paramètres positionnels ($@, $0)](with-dollar-at.md)** : Fragmenter un mot-clé bloqué en insérant des expansions de paramètres positionnels qui s'évaluent à rien, who$@ami, who${@}ami, et abuser de $0 comme invocation de shell,...
- **[Substitution de commande vide ($())](with-dollar-parentheses.md)** : Fragmenter un mot-clé bloqué en insérant une substitution de commande $() vide, who$()ami, pour que le shell n'exécute rien, la supprime et réassemble le mot-clé, con...
- **[Guillemets doubles](with-double-quote.md)** : Fragmenter un mot-clé de commande bloqué avec des paires de guillemets doubles vides, w\"h\"o\"a\"m\"i, pour que le shell retire les guillemets et réassemble le mot, contournant une blocklist litté...
- **[Guillemets simples](with-single-quote.md)** : Fragmenter un mot-clé de commande bloqué avec des paires de guillemets simples vides, w'h'o'a'm'i, pour que le shell retire les guillemets et réassemble le mot, contournant une blocklist litté...

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à mot-clé fragmenté.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
