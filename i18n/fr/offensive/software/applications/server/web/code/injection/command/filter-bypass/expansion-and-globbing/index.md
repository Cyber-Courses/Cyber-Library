---
order: 3
title: "Contournement par expansion et globbing"
description: "Reconstruire des commandes et des chemins via l'expansion d'accolades, le globbing par caractères génériques, l'expansion de tilde et l'expansion de variables."
keywords:
  - expansion d'accolades
  - caractère générique
  - globbing
  - expansion de tilde
  - expansion de variables
---

# Expansion et globbing

Reconstruisez commandes et chemins via l'expansion du shell plutôt que du texte littéral : l'expansion d'accolades, le globbing par caractères génériques (`/???/c?t /???/p?sswd`), l'expansion de tilde et l'expansion de variables ou de paramètres assemblent la cible à partir de caractères que la liste de blocage ne surveille pas.

## Pages

- **[Reconstruction de caractères](character-reconstruction.md)** : reconstruire des caractères interdits comme / à partir de sous-chaînes de variables d'environnement (${HOME:0:1}), de la traduction tr et de printf, pour qu'un filtre bloquant des caractères spécifiqu...
- **[Expansion d'accolades](with-brace-expansion.md)** : utiliser l'expansion d'accolades de Bash, {cat,/etc/passwd}, {ls,-la}, pour scinder une commande en jetons séparés par des virgules ne transportant aucun espace ni mot-clé filtré, pour qu'une l...
- **[Expansion de tilde](with-tilde-expansion.md)** : reconstruire des chemins filtrés à partir de l'expansion de tilde du shell, ~ pour HOME, ~+ pour PWD, ~- pour OLDPWD, pour que les préfixes de répertoire soient produits par le shell plutôt que...
- **[Expansion de variables et de paramètres](with-variable-expansion.md)** : assembler des commandes à partir de l'expansion de paramètres du shell, ${PATH:0:1} pour /, ${IFS} pour les espaces, découpage de sous-chaînes et substitution de motifs, pour que les caractères interdits...
- **[Caractères génériques](with-wildcards.md)** : reconstruire des noms de binaires et des chemins filtrés avec les caractères glob du shell, /???/c?t /???/p?sswd, /bin/c*, pour que le littéral bloqué n'apparaisse jamais dans la requête...

## Outils

- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements basés sur l'expansion.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour concevoir des payloads d'expansion et de glob.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
