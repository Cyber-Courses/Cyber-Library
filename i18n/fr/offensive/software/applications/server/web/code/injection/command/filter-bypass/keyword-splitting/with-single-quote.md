---
order: 2
title: "Fragmentation de mots-clés avec guillemets simples : w'h'o'a'm'i"
description: "Fragmenter un mot-clé de commande bloqué avec des paires de guillemets simples vides, w'h'o'a'm'i, pour que le shell retire les guillemets et réassemble le mot, contournant une blocklist littérale."
keywords:
  - injection de commande
  - fragmentation de mots-clés
  - guillemets simples
  - contournement de filtre
  - retrait de guillemets
  - contournement de WAF
---

# Guillemets simples

Les guillemets simples dans le shell délimitent une chaîne littérale, mais une **paire vide de guillemets simples** (`''`) ne délimite rien. Durant le retrait des guillemets, le shell supprime les guillemets et concatène ce qui les entoure en un seul mot. Parsemer `''` à travers un mot-clé filtré, `w'h'o'a'm'i`, se réassemble donc en `whoami` au moment de l'exécution sans jamais apparaître comme le mot-clé contigu dans l'entrée brute.

## Pourquoi le shell retire les guillemets

Le retrait des guillemets est une étape définie de l'expansion du shell qui s'exécute après que le filtre a déjà inspecté les octets. `w''h''o''a''m''i` se tokenise en un seul mot ; chaque `''` contribue un segment littéral vide, et le shell retire tous les caractères guillemets, joignant les segments en `whoami`. Une blocklist correspondant à la chaîne littérale `whoami` ne voit que la forme entre guillemets et la laisse passer. Les guillemets doivent rester **équilibrés**, un guillemet simple impair laisse le shell en attente d'un guillemet fermant au lieu d'exécuter la commande.

## Payloads

Paires vides entre les caractères, ou juste assez pour briser la signature :

```
w''h''o''a''m''i
who''ami
c'a't /etc/passwd
i'd'
```

Vous pouvez aussi entourer des caractères uniques de guillemets non vides, ce qui est équivalent :

```
'whoami'          # one quoted word, still runs whoami
```

## Dans une requête HTTP

Les guillemets simples passent couramment sans encodage ; encodez en URL comme `%27` au besoin :

```
w%27h%27oami
```

## Combinaison avec d'autres évasions

La fragmentation par guillemets ne dissimule que le mot-clé. Ajoutez un substitut d'espace ou un séparateur là où ceux-ci sont aussi filtrés :

```
c'a't${IFS}/etc/passwd
127.0.0.1;w'h'o'a'm'i
```

## Contexte et mises en garde

- Il s'agit d'une technique **shell** (`sh -c`, `system()`, backticks).
- Les guillemets doivent être **équilibrés**. Un nombre impair de guillemets simples laisse un littéral ouvert et le shell attend un guillemet fermant, cassant la commande, appariez-les toujours.
- Si votre point d'injection se situe déjà **à l'intérieur** de guillemets simples (par ex. `sh -c 'ping '$x''`), vous devez d'abord fermer ce littéral avec un `'` avant que cette astuce ne s'applique ; voir la discussion sur le contexte de guillemets sur la page des métacaractères du shell.
- Les guillemets simples sont plus forts que les guillemets doubles : à l'intérieur de `'...'` aucune expansion ne se produit, donc cette forme sert uniquement à la fragmentation, pas à déclencher une substitution.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à guillemets fragmentés.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
