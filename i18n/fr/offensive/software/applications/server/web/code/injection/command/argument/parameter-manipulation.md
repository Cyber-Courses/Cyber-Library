---
title: "Injection d'argument et manipulation de paramètre sans shell"
description: "Transformer une valeur utilisateur en argv pour qu'un binaire appelé l'analyse comme un flag, sans shell, plus les astuces de worst-fit et de caractères fullwidth et le concept de contournement d'escapeshellarg/escapeshellcmd."
keywords:
  - injection d'argument
  - manipulation de paramètre
  - injection argv
  - mapping worst-fit
  - contournement d'escapeshellarg
  - caractères fullwidth
---

# Manipulation de paramètre

Lorsqu'une application lance un **binaire fixe** avec un tableau d'arguments et sans shell, les métacaractères (`;`, `|`, `$()`) sont inertes. L'injection d'argument exploite une couture différente : la valeur utilisateur atterrit dans `argv` et le **binaire lui-même la ré-analyse comme une option** plutôt que comme des données. Aucun nouveau processus n'est lancé, et pourtant le processus existant est orienté vers la lecture de fichiers, l'écriture de fichiers ou l'exécution de sous-programmes qu'il prend nativement en charge.

## Mécanisme

```python
# user_arg is attacker-controlled; a fixed URL follows it in argv
subprocess.run(["curl", user_arg, "https://report.internal/collect"], shell=False)
```

Pas de shell, donc `;id` ne fait rien. Mais une valeur `-o/var/www/html/x.php` est analysée par `curl` comme le flag de sortie `-o`, de sorte que la réponse de l'URL fixe `https://report.internal/collect` est **écrite** dans la racine web, une récupération devient une écriture. L'URL compte : `-o <path>` sans URL dans argv échoue simplement avec « no URL specified », donc cette primitive a besoin d'une URL présente (fournie ici par l'argument fixe en fin, ou par un second flag comme `--url`). La cause racine est que les **données positionnelles et les options partagent le même espace argv**, et que la plupart des parseurs acceptent les options n'importe où sur la ligne. Tout jeton commençant par `-` (ou `@` pour certains outils) est un flag candidat.

## Fournir un flag au lieu de données

Le gain dépend de la grammaire d'options du binaire cible :

```
-o/var/www/html/p.php        # curl: write to web root
-K/etc/passwd                # curl: parse an attacker-named file as a config (one argv element)
--use-askpass=/tmp/x.sh      # wget: command execution
-c core.sshCommand=id        # git: run a command during clone/fetch
--checkpoint-action=exec=id  # tar: command execution
```

Utilisez la forme `--flag=value` pour que la valeur tienne dans un seul jeton lorsque l'application ajoute un argument fixe après le vôtre. [GTFOBins](https://gtfobins.github.io/) catalogue ces primitives par binaire.

## Astuces de worst-fit / caractères fullwidth

Certains runtimes convertissent l'Unicode en ANSI avec un **mapping best-fit / worst-fit** avant de remettre argv à un binaire natif. Un caractère qui n'est pas un tiret peut être rétro-converti en un tiret, glissant un flag au-delà d'un filtre qui ne bloque que l'ASCII `-`. Le cas Windows classique mappe les codepoints fullwidth et sosies vers l'ASCII :

```
U+FF0D  FULLWIDTH HYPHEN-MINUS   →  -
U+2010  HYPHEN                    →  -
U+FF4F  FULLWIDTH LATIN O         →  o
```

Ainsi une valeur comme `？-ｏС:\path` (caractères fullwidth) peut arriver au processus lancé sous la forme `-oC:\path` après conversion worst-fit, réintroduisant une option que l'application croyait avoir retirée. Le même mapping affecte les caractères de guillemet et d'espace, permettant un découpage d'arguments là où le lanceur croyait avoir passé un seul jeton. C'est le mécanisme derrière plusieurs découvertes de contrebande d'arguments `php-cgi`/`CreateProcess` sous Windows.

## Concept de contournement d'escapeshellarg / escapeshellcmd

Les deux échappeurs de PHP protègent des choses différentes, et les confondre laisse un trou :

- `escapeshellcmd()` neutralise les métacaractères de shell sur l'**ensemble de la chaîne de commande** mais ne met **pas** entre guillemets les arguments individuels, de sorte qu'il stoppe une nouvelle commande mais laisse tout de même une valeur être lue comme un **flag** (`-o`, `--config`). Ce n'est pas une défense contre l'injection d'argument.
- `escapeshellarg()` enveloppe un argument de guillemets pour le garder comme un seul jeton positionnel. Un mauvais usage, échapper le mauvais segment, ou concaténer une valeur échappée à côté d'un `-` non échappé, rouvre l'injection de flag.

La leçon offensive : un sink qui appelle `escapeshellcmd()` (courant, car il « ressemble » à la bonne fonction) est fréquemment encore vulnérable à l'injection d'argument même si l'injection classique de métacaractères est bloquée. Sondez avec des valeurs à tiret initial quel que soit l'échappement visible.

## Trouver la frontière

Le correctif canonique du défenseur est un séparateur `--` (« tout ce qui suit est positionnel »). Si votre entrée est placée **avant** tout `--`, les flags sont en jeu ; si elle est après, vous êtes limité à l'abus positionnel, à la path traversal, à l'inclusion `@file` ou à la contrebande de protocole (`file://`, `gopher://`), plutôt qu'à l'injection de flag.

## Outils

- **[GTFOBins](https://gtfobins.github.io/)**, flags de lecture/écriture de fichiers et d'exécution de commandes par binaire.
- **[Burp Suite](https://portswigger.net/burp)** Repeater/Intruder pour fuzzer un emplacement d'entrée avec des flags candidats.
- Des copies locales des binaires cibles pour confirmer la sémantique attaché contre séparé des options et la gestion de `--`.

## Références

- [CWE-88: Argument Injection](https://cwe.mitre.org/data/definitions/88.html)
- [PortSwigger Research: Argument injection](https://portswigger.net/research)
- [GTFOBins](https://gtfobins.github.io/)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
