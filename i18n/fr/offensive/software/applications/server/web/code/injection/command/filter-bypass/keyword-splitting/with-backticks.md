---
order: 1
title: "Fragmentation de mots-clés avec backticks vides : briser des commandes bloquées"
description: "Fragmenter un mot-clé de commande filtré avec une substitution de backticks vide, wh``oami, pour que le shell supprime la commande vide et réassemble le mot-clé, déjouant une blocklist littérale."
keywords:
  - injection de commande
  - fragmentation de mots-clés
  - backticks
  - contournement de filtre
  - substitution de commande
  - contournement de WAF
---

# Backticks vides

Une blocklist qui correspond à un mot-clé littéral tel que `whoami` ou `cat` peut être déjouée en insérant une **substitution de commande vide** à l'intérieur du mot. Une paire vide de backticks (`` `` ``) exécute « rien », s'évalue à la chaîne vide, et est supprimée durant l'expansion de mots, laissant les caractères environnants se rejoindre pour former le mot-clé original. Le filtre voit `` wh``oami `` ; le shell exécute `whoami`.

## Pourquoi le shell réassemble le mot

Les backticks sont une substitution de commande héritée : `` `cmd` `` exécute `cmd` et insère sa sortie standard dans la ligne de commande. Une paire vide exécute une commande vide, qui ne produit ni sortie ni erreur, et se substitue à rien. Fait crucial, la substitution se produit *à l'intérieur* d'un seul mot, donc les octets de part et d'autre sont concaténés après que le résultat vide est retiré. `` wh``oami `` se tokenise en l'unique mot `whoami`, que le shell recherche ensuite comme commande. Un filtre à chaîne littérale inspectant l'entrée brute ne voit jamais le mot-clé contigu.

## Payloads

```
wh``oami
ca``t /etc/passwd
i``d
```

La même idée fragmente n'importe quelle sous-chaîne bloquée, placée n'importe où dans le mot :

```
who``ami
``whoami
whoami``
```

## Dans une requête HTTP

Les backticks passent généralement sans encodage, mais encodez-les si l'application les altère (`` ` `` est `%60`) :

```
wh%60%60oami
```

## Combinaison avec d'autres évasions

La fragmentation par backticks ne dissimule que le mot-clé. Si les espaces sont aussi filtrés, ajoutez un substitut d'espace ; si les séparateurs sont filtrés, associez un saut de ligne :

```
ca``t${IFS}/etc/passwd
127.0.0.1%0awh``oami
```

## Formes apparentées

Le même principe « insérer quelque chose qui s'évalue à rien » fonctionne aussi avec une substitution `$()` vide (`who$()ami`), des guillemets vides (`who''ami`) et des antislashs (`who\ami`). Les backticks sont simplement la plus ancienne syntaxe de substitution et sont honorés par tous les shells POSIX, y compris le `/bin/sh` minimal.

## Notes de contexte

- Il s'agit d'une technique **shell** ; la chaîne doit atteindre `sh -c`, `system()`, ou une exécution par backticks.
- À l'intérieur de guillemets doubles, les backticks déclenchent toujours la substitution, donc `` "wh``oami" `` se réassemble aussi, utile quand votre injection atterrit dans un contexte entre guillemets.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à backticks vides.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
