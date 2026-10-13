---
order: 6
title: "Fragmentation de mots-clés avec substitution de commande vide : who$()ami"
description: "Fragmenter un mot-clé bloqué en insérant une substitution de commande $() vide, who$()ami, pour que le shell n'exécute rien, la supprime et réassemble le mot-clé, contournant une blocklist littérale."
keywords:
  - injection de commande
  - fragmentation de mots-clés
  - substitution de commande
  - "$()"
  - contournement de filtre
  - contournement de WAF
---

# Substitution de commande vide ($())

La substitution de commande moderne utilise `$( ... )`. Une substitution **vide**, `$()`, n'exécute aucune commande, ne produit aucune sortie, et est retirée durant l'expansion, de sorte que, comme des guillemets ou des backticks vides, elle peut être glissée à l'intérieur d'un mot-clé filtré pour briser la signature. `who$()ami` exécute une commande vide entre `who` et `ami`, puis se réduit à `whoami`.

## Pourquoi le shell réassemble le mot

`$(cmd)` exécute `cmd` et substitue sa sortie standard. Sans rien entre les parenthèses, il n'y a rien à exécuter ; la substitution produit la chaîne vide. Parce qu'elle se produit à l'intérieur d'un seul mot, les octets environnants se concatènent une fois le résultat vide inséré. `who$()ami` se tokenise en l'unique mot `whoami`, que le shell résout ensuite comme commande. Une blocklist littérale inspectant les octets bruts ne voit jamais le mot-clé contigu.

## Payloads

```
who$()ami
ca$()t /etc/passwd
i$()d
```

L'insertion peut aller n'importe où dans le mot, et plus d'une fois :

```
$()whoami
who$()a$()mi
```

## `$()` versus backticks

`$()` et `` `` `` sont équivalents à cette fin ; `$()` est préféré car il s'imbrique proprement et est moins susceptible d'être altéré par un traitement intermédiaire. Les deux sont des astuces « insérer quelque chose qui s'évalue à rien », de la même famille que `''`, `""`, `\` et `$@`.

## Dans une requête HTTP

`$`, `(`, `)` passent généralement sans encodage ; encodez si nécessaire (`$`=`%24`, `(`=`%28`, `)`=`%29`) :

```
who%24%28%29ami
```

## Combinaison avec d'autres évasions

La substitution ne dissimule que le mot-clé. Ajoutez un substitut d'espace ou un séparateur là où ceux-ci sont filtrés :

```
ca$()t${IFS}/etc/passwd
127.0.0.1;who$()ami
```

## Contexte et mises en garde

- Il s'agit d'une technique **shell** (`sh -c`, `system()`, backticks) ; dans un appel `argv` pur, `$()` est une chaîne littérale.
- La substitution `$()` se déclenche aussi à l'intérieur de guillemets **doubles**, donc `"who$()ami"` se réassemble également, pratique quand votre injection atterrit dans un contexte entre guillemets doubles.
- Un `$()` **non vide** est la primitive d'exécution elle-même (`$(id)`) ; ici la forme vide est utilisée uniquement pour la dissimulation.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à substitution vide.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
