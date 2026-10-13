---
order: 10
title: "Injection de log"
description: "De l'entrée non fiable écrite dans des lignes ou des champs de log, laissant un attaquant forger des événements, décaler des champs, et confondre les parseurs, agrégateurs et pipelines SIEM qui lisent les logs en aval."
keywords:
  - injection de log
  - log forging
  - injection CRLF
  - log poisoning
  - évasion de SIEM
---

# Log

L'injection de log survient lorsque de l'entrée contrôlée par l'attaquant est écrite dans une ligne de log ou un champ de log structuré sans que le logger neutralise les caractères qui définissent les frontières d'enregistrement, les délimiteurs, ou la structure d'objet.

Parce que la plupart du code de logging est une fine concaténation de chaînes autour de ce que la requête a fourni, un attaquant qui contrôle un nom d'utilisateur, un en-tête, un chemin ou un paramètre de query contrôle souvent des octets que le format de log traite comme de la structure plutôt que du contenu. Le gain n'est généralement pas l'exécution de code dans le logger lui-même mais le contrôle de l'enregistrement : forger des entrées de log entières, usurper les champs de sévérité et de source, incriminer un autre compte, casser la regex ou le schéma dont dépend un parseur en aval, et faire passer en fraude du balisage qui se déclenche quand un analyste ouvre le log dans une console web. L'entrée loggée peut aussi atteindre des sinks de lookup qui l'évaluent, ce qui est hors de portée ici. Ce sous-arbre est organisé par le format de log cassé : newline forging contre les logs orientés ligne, injection de champ délimité contre les logs CSV, TSV et `key=value`, et injection de log JSON contre les pipelines structurés alimentant ELK ou Loki.

## Pages

- **[Delimited field injection](delimited-field-injection.md)** : injecter le caractère délimiteur ou de guillemet d'un format de log structuré pour que l'entrée loggée décale des colonnes, ajoute de faux champs, ou casse les parseurs regex qui con...
- **[JSON log injection](json-log-injection.md)** : lorsqu'une ligne de log JSON est assemblée par concaténation de chaînes, injecter un guillemet et une virgule pour ajouter ou écraser des clés comme level, user et trace, casser l'agréga...
- **[Newline forging](newline-forging.md)** : injecter un CRLF ou un saut de ligne nu dans l'entrée loggée pour qu'une seule instruction de log émette deux lignes, forgeant un second enregistrement de log avec timestamp, sév...

## Outils

- **Burp Repeater** : forger des payloads CRLF, de délimiteur et de champ JSON dans l'entrée loggée.
- **curl** : envoyer des payloads d'injection de log forgés vers des paramètres de requête.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
