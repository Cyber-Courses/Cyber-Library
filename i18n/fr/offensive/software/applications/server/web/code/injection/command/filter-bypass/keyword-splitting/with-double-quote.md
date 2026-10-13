---
order: 3
title: "Fragmentation de mots-clés avec guillemets doubles : w\"h\"o\"a\"m\"i"
description: "Fragmenter un mot-clé de commande bloqué avec des paires de guillemets doubles vides, w\"h\"o\"a\"m\"i, pour que le shell retire les guillemets et réassemble le mot, contournant une blocklist littérale."
keywords:
  - injection de commande
  - fragmentation de mots-clés
  - guillemets doubles
  - contournement de filtre
  - retrait de guillemets
  - contournement de WAF
---

# Guillemets doubles

Comme les guillemets simples, une **paire vide de guillemets doubles** (`""`) ne délimite rien et est retirée durant le retrait des guillemets, concaténant ses voisins en un seul mot. Insérer `""` à travers un mot-clé filtré, `w"h"o"a"m"i`, se réassemble en `whoami` à l'exécution tout en dissimulant le mot-clé contigu à une blocklist littérale.

## Pourquoi le shell retire les guillemets

Le retrait des guillemets s'exécute comme une étape d'expansion définie après le filtrage. `w""h""o""a""m""i` est un seul mot ; chaque `""` est un segment vide, et le shell supprime chaque caractère guillemet et joint les morceaux en `whoami`. Une blocklist correspondant au mot-clé littéral ne le voit jamais. Gardez les guillemets **équilibrés**, un nombre impair laisse une chaîne non terminée et le shell signale une erreur de syntaxe au lieu d'exécuter la commande.

## Payloads

```
w""h""o""a""m""i
who""ami
c"a"t /etc/passwd
i"d"
```

## Différence avec les guillemets simples

Les guillemets doubles sont « faibles » : le shell effectue toujours l'expansion de variables, la substitution de commande et le traitement de l'antislash à l'intérieur. Pour la *fragmentation*, cela ne fait aucune différence, les paires vides disparaissent de la même façon. Mais cela signifie aussi que les guillemets doubles peuvent faire plus que les guillemets simples dans une injection :

```
"$(id)"          # substitution still fires inside double quotes
"`whoami`"       # backticks still fire inside double quotes
```

Ainsi la fragmentation par guillemets doubles est interchangeable avec la fragmentation par guillemets simples pour dissimuler un mot-clé, et se compose en outre avec la substitution lorsque vous avez besoin d'exécution plutôt que de simple dissimulation.

## Dans une requête HTTP

Encodez le guillemet en URL comme `%22` si l'application ne l'accepte pas brut :

```
w%22h%22oami
```

## Combinaison avec d'autres évasions

Ajoutez un substitut d'espace ou un séparateur là où ceux-ci sont filtrés :

```
c"a"t${IFS}/etc/passwd
127.0.0.1;w"h"o"a"m"i
```

## Contexte et mises en garde

- Il s'agit d'une technique **shell** (`sh -c`, `system()`, backticks).
- Les guillemets doivent être **équilibrés** ; un `"` non apparié laisse une chaîne ouverte et casse la commande.
- Si votre injection se situe déjà à l'intérieur d'un contexte entre guillemets doubles, rappelez-vous que `$`, `` ` `` et `\` y restent actifs, utile pour s'évader, mais cela signifie aussi qu'un `$` ou un backtick égaré dans votre payload peut être interprété.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à guillemets fragmentés.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
