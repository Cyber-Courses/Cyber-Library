---
order: 4
title: "Contournement par encodage et obfuscation"
description: "Dissimuler le payload par décodage hexadécimal/ANSI-C à l'exécution et par variation de casse contre des listes de blocage sensibles à la casse."
keywords:
  - encodage hexadécimal
  - quoting ANSI-C
  - casse aléatoire
  - obfuscation
  - contournement de WAF
---

# Encodage et obfuscation

Dissimulez le payload aux analyseurs naïfs et aux listes de blocage sensibles à la casse : décodez des séquences hexadécimales ou ANSI-C à l'exécution pour que le mot-clé n'apparaisse jamais dans la requête, ou variez la casse lorsque le shell cible est insensible à la casse, de sorte que les octets inspectés ne correspondent pas à la commande qui s'exécute.

## Pages

- **[Hexadécimal et décodage à l'exécution](hex-encoding.md)** : transporter une commande sous forme d'hexadécimal ou d'échappements ANSI-C, $'\\x77\\x68\\x6f\\x61\\x6d\\x69', echo -e, xxd -r, et la décoder à l'exécution pour que le mot-clé filtré n'appara...
- **[Casse aléatoire](with-random-case.md)** : contourner les listes de blocage sensibles à la casse en variant la capitalisation, wHoAmi, CeRTutil, là où l'interpréteur est insensible à la casse (Windows cmd, PowerShell) pour que la comm...

## Outils

- **[commix](https://github.com/commixproject/commix)** : les modules tamper automatisent les contournements par encodage et par casse.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Decoder pour concevoir des payloads encodés.

## Références

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
