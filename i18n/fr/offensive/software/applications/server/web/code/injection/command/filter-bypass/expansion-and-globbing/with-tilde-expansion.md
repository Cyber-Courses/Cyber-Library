---
order: 3
title: "Contournement de filtre d'injection de commande par expansion de tilde"
description: "Reconstruire des chemins filtrés à partir de l'expansion de tilde du shell, ~ pour HOME, ~+ pour PWD, ~- pour OLDPWD, pour que les préfixes de répertoire soient produits par le shell plutôt que tapés comme des littéraux filtrés."
keywords:
  - injection de commande
  - expansion de tilde
  - contournement de filtre
  - reconstruction de chemin
  - PWD
  - OLDPWD
---

# Expansion de tilde

L'expansion de tilde remplace un `~` de tête par un chemin de répertoire avant que la commande ne s'exécute. Un `~` simple devient `$HOME`, `~+` devient `$PWD` (le répertoire courant) et `~-` devient `$OLDPWD` (le répertoire précédent). Parce que le shell produit lui-même le chemin, un attaquant peut reconstruire un préfixe de système de fichiers sans taper le nom de répertoire littéral que surveille une liste de blocage.

## Pourquoi le shell le normalise

L'expansion de tilde se produit tôt dans la séquence d'expansion du shell, aux côtés de l'expansion d'accolades et de l'expansion de paramètres, avant le découpage en mots et avant l'exécution. Un mot commençant par `~`, `~+` ou `~-` est réécrit en la chaîne de répertoire correspondante. Le filtre qui inspecte la requête ne voit que le jeton tilde ; le shell remet à la commande un chemin pleinement qualifié. Le littéral interdit (par exemple un préfixe de répertoire personnel, ou un chemin qui contiendrait autrement un mot-clé bloqué) est fabriqué à partir de l'état du shell plutôt que fourni par l'attaquant.

## Payloads

`~+` se développe en le répertoire de travail courant, ce qui est utile lorsqu'un chemin relatif est filtré ou lorsque vous avez besoin d'un préfixe absolu sans taper `/` :

```
cat ~+/config/secret.env
~+/uploaded_binary
```

`~` atteint le répertoire personnel du compte de service, souvent là où résident la configuration de l'application, les clés SSH ou les fichiers d'historique, sans le nommer :

```
cat ~/.ssh/id_rsa
cat ~/.bash_history
ls -la ~
```

`~-` référence `OLDPWD` ; lorsqu'une application change de répertoire entre les opérations, il peut pointer vers un emplacement précédemment visité :

```
cat ~-/.env
```

Le tilde d'utilisateur nommé (`~user`) se développe en le répertoire personnel de ce compte d'après `/etc/passwd`, ce qui vous permet de cibler les fichiers d'un autre utilisateur sans chemin littéral :

```
cat ~www-data/.config
ls ~root 2>/dev/null
```

Le tilde se combine aux astuces sans espace, puisque la technique ne fournit que le préfixe de répertoire et qu'il vous faut encore des séparateurs :

```
cat${IFS}~/.ssh/id_rsa
{cat,~/.bash_history}
```

Il s'associe également au découpage de variables pour reconstruire une barre oblique de tête ailleurs dans la commande tandis que `~+` gère le préfixe du répertoire de travail.

## Notes opérationnelles

- L'expansion de tilde est une fonctionnalité de Bash et de la plupart des shells interactifs ; confirmez que le sink lance un shell qui l'effectue. L'expansion ne s'applique qu'à un `~` de tête **non quoté**, à l'intérieur de guillemets c'est un littéral.
- `~+`/`~-` dépendent de `PWD`/`OLDPWD` qui doivent être définis, ce qu'ils sont normalement dans un shell lancé avec un environnement. La valeur reflète l'endroit où s'exécute le processus enfant de l'application.
- La technique reconstruit des **préfixes de répertoire**, pas des noms de binaires, associez-la au globbing ou à l'expansion de variables lorsque la commande ou les caractères interdits eux-mêmes sont filtrés.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour concevoir des payloads d'expansion de tilde.
- Test manuel avec Burp Repeater et des payloads conçus à la main.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Tilde Expansion](https://www.gnu.org/software/bash/manual/html_node/Tilde-Expansion.html)
