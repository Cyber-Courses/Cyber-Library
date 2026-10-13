---
order: 2
title: "Injection et falsification : entrées falsifiées et abus des parseurs en aval"
description: "L'ingestion syslog fait confiance au contenu des messages, si bien qu'un attaquant injecte des entrées falsifiées et, en intégrant des retours à la ligne et des délimiteurs, scinde un enregistrement en plusieurs ou casse l'analyse des champs, empoisonnant la corrélation du SIEM. Là où le pipeline achemine des valeurs journalisées vers un puits vulnérable, un tableau de bord, un parseur ou une application en aval, le contenu injecté exploite aussi ce puits."
keywords:
  - injection de logs
  - injection de retour à la ligne
  - falsification
  - empoisonnement de siem
  - log4shell
---

# Injection et falsification

Parce que le pipeline syslog fait confiance au contenu des messages, un attaquant capable d'émettre des enregistrements (directement vers le collecteur, ou indirectement en amenant une application surveillée à journaliser des données contrôlées par l'attaquant) forge et manipule l'enregistrement. La falsification pure implante de faux évènements. Plus subtilement, intégrer des caractères de retour à la ligne et des délimiteurs dans une valeur journalisée scinde un enregistrement en plusieurs évènements apparents ou casse l'analyse des champs du collecteur, si bien qu'un attaquant peut fabriquer des lignes de log supplémentaires, désynchroniser l'analyse structurée, et empoisonner les règles de corrélation et les tableaux de bord qui consomment le flux. Et là où les valeurs journalisées affluent vers un puits vulnérable, une visionneuse de logs qui les affiche, un parseur qui les évalue, ou une application en aval, le contenu injecté exploite ce puits ; la classe Log4Shell, où une chaîne journalisée déclenchait une recherche JNDI, est l'exemple extrême de données d'attaquant dans un log devenant une exécution de code.

```bash
# forge an event, and split a record with an embedded newline to fake extra lines
printf '<34>app: user=admin action=login\n<34>app: CRITICAL all clear\n' \
  | nc -u -w1 <collector> 514
# indirect: make a monitored app log attacker-controlled data that carries
# newlines/delimiters (e.g. a crafted username, User-Agent, or URL) to inject
# downstream; if the sink evaluates logged values, injection reaches that sink
```

## Notes d'exploitation

- L'injection de retour à la ligne/CRLF et de délimiteurs est la technique centrale : elle transforme une valeur journalisée en plusieurs enregistrements ou corrompt l'analyse des champs structurés, permettant à un attaquant de fabriquer des évènements et de désynchroniser le SIEM sans jamais usurper un paquet.
- L'injection indirecte est souvent le vecteur réaliste : les champs contrôlés par l'attaquant que les applications journalisent (noms d'utilisateurs, en-têtes, URL, messages d'erreur) portent la charge, de sorte que l'attaquant n'a jamais besoin de parler syslog directement.
- Le plus fort impact est un puits en aval vulnérable : si les valeurs journalisées sont affichées, analysées ou évaluées de façon non sûre, l'injection devient un XSS stocké dans un tableau de bord de logs, une exploitation de parseur, ou une exécution de code (le motif Log4Shell) ; évaluez ce qui consomme les logs.
- À associer avec l'[usurpation de source](source-spoofing.md) pour contrôler à la fois l'origine et le contenu et produire des évènements falsifiés convaincants.

## Références

- [OWASP: log injection](https://owasp.org/www-community/attacks/Log_Injection)
- [RFC 5424 (syslog message format)](https://datatracker.ietf.org/doc/html/rfc5424)
