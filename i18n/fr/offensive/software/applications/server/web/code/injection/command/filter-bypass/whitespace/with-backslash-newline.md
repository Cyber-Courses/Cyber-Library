---
order: 3
title: "Continuation de ligne antislash-saut de ligne pour fragmenter une commande sur plusieurs lignes"
description: "Utiliser un antislash suivi d'un saut de ligne pour briser un mot-clé ou une commande à travers des lignes physiques pour qu'une blocklist littérale le manque tandis que le shell rejoint et l'exécute."
keywords:
  - injection de commande
  - continuation de ligne
  - antislash saut de ligne
  - contournement de filtre
  - fragmentation de mots-clés
---

# Continuation antislash + saut de ligne

Un antislash immédiatement suivi d'un saut de ligne est une **continuation de ligne** : le shell retire les deux caractères et joint les deux lignes physiques en une seule ligne logique avant tout autre parsing. Cela donne à un attaquant un moyen de fragmenter un mot-clé filtré, ou une commande entière, à travers des lignes pour qu'aucun token bloqué n'apparaisse de manière contiguë dans l'entrée brute, tandis que le shell le réassemble silencieusement à l'exécution.

## Pourquoi le shell efface la coupure

Durant la tokenisation, le shell traite `\<newline>` en premier, supprimant la paire entièrement. La séquence `who\<newline>ami` devient donc l'unique mot `whoami` *avant* que le shell ne décide ce qui est un nom de commande. Un filtre qui correspond à la chaîne littérale `whoami`, ou qui inspecte chaque ligne indépendamment, ne voit que `who` et `ami` et laisse passer la requête.

## Payloads

Dans une requête HTTP, le saut de ligne est encodé en URL comme `%0a` ; l'antislash est `%5c` ou un `\` littéral :

```
who\%0aami
id\%0a
\%0awhoami
```

En octets bruts, le payload est un antislash en fin de ligne :

```
who\
ami
```

Le shell joint ceux-ci en `whoami`.

## Fragmenter une commande complète et ses arguments

La continuation peut casser n'importe quelle position, y compris à l'intérieur d'un chemin, vous permettant de dissimuler `cat /etc/passwd` à une signature :

```
ca\
t /et\
c/pa\
sswd
```

Chaque antislash-saut de ligne est supprimé, produisant `cat /etc/passwd`.

## Combinaison avec des séparateurs

La continuation ne dissimule qu'un mot-clé ; vous devez toujours démarrer la deuxième commande. Associez-la à un terminateur par saut de ligne ou à un séparateur autorisé, et à un substitut d'espace si les espaces sont bloqués :

```
127.0.0.1%0awho\%0aami
127.0.0.1;ca\%0at${IFS}/etc/passwd
```

## Notes de contexte

- Il s'agit d'une technique **shell** : la chaîne doit atteindre `sh -c`, `system()`, ou les backticks, où la continuation est honorée.
- La continuation par antislash diffère de la simple fragmentation de mots-clés par antislash (`w\ho\ami`) : là l'antislash échappe le *caractère* suivant ; ici il échappe spécifiquement le *saut de ligne* pour fusionner les lignes.
- `cmd.exe` utilise `^` en fin de ligne pour la continuation plutôt que `\`.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads de continuation de ligne.
- Tests manuels avec Burp Repeater et des payloads élaborés.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
