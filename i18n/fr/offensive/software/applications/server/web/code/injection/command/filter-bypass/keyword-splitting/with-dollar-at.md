---
order: 5
title: "Fragmentation de mots-clés avec paramètres positionnels : who$@ami et $0"
description: "Fragmenter un mot-clé bloqué en insérant des expansions de paramètres positionnels qui s'évaluent à rien, who$@ami, who${@}ami, et abuser de $0 comme invocation de shell, pour qu'une blocklist littérale manque le mot-clé."
keywords:
  - injection de commande
  - fragmentation de mots-clés
  - paramètres positionnels
  - "$@"
  - "$0"
  - contournement de filtre
---

# Paramètres positionnels ($@, $0)

Les paramètres spéciaux du shell s'évaluent à rien dans un contexte non interactif sans arguments, ce qui les rend utiles comme séparateur invisible à l'intérieur d'un mot. `$@` (tous les paramètres positionnels) et `$*` s'évaluent à la chaîne vide quand aucun n'est défini, de sorte que `who$@ami` se réassemble en `whoami` après expansion. `$0` est utile séparément : il contient le nom du shell lui-même et peut être exécuté directement.

## `$@` et `$*` comme séparateur vide

Dans `sh -c "..."` sans arguments supplémentaires, `$@` s'évalue à rien. Placé à l'intérieur d'un mot-clé, il disparaît et les voisins se joignent :

```
who$@ami
who${@}ami
c$@at /etc/passwd
i$@d
```

La forme à accolades `${@}` est plus robuste car elle délimite explicitement le nom du paramètre, empêchant le shell de lire les lettres suivantes comme en faisant partie. Une blocklist correspondant à `whoami` ne voit que `who$@ami`.

## Pourquoi cela s'évalue à vide

`$@` est la liste des arguments positionnels `$1 $2 ...`. Un sink web qui appelle `sh -c "<string>"` ne passe aucun paramètre positionnel, donc la liste est vide et l'expansion contribue zéro caractère au mot. Il en va de même pour `$*`, `$1`, `$9` et autres positionnels non définis.

## `$0` comme accès au shell

`$0` s'évalue au nom sous lequel le shell a été invoqué (typiquement `sh` ou `bash`). Cela en fait un moyen, exempt de mot-clé filtré, de lancer un shell ou d'y acheminer une commande :

```
echo id | $0
$0 -c id
```

Ici aucun mot-clé littéral `sh`/`bash` n'apparaît dans le payload, pourtant `$0` résout vers l'interpréteur et exécute la commande préparée.

## Dans une requête HTTP

`$`, `@`, `{`, `}` passent généralement sans encodage ; encodez au besoin (`$`=`%24`, `@`=`%40`) :

```
who%24%40ami
```

## Combinaison avec d'autres évasions

Associez un substitut d'espace ou un séparateur :

```
c$@at${IFS}/etc/passwd
127.0.0.1;who${@}ami
```

## Contexte et mises en garde

- Il s'agit d'une technique **shell** (`sh -c`, `system()`, backticks) ; dans un appel `argv` pur, `$@` est une chaîne littérale.
- Si le sink passe au shell des arguments positionnels contrôlés par l'utilisateur, `$@`/`$1` peuvent ne pas être vides, préférez un positionnel connu pour être non défini tel que `$9`.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à paramètres positionnels.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
