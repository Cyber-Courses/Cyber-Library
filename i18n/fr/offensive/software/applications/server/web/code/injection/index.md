---
order: 5
title: "Vulnérabilités d'injection"
description: "Des données non fiables liées à quelque chose que le serveur exécute, analyse, stocke, rend ou requête, organisées selon la primitive dangereuse attaquée."
keywords:
  - injection
  - injection de commande
  - injection SQL
  - injection NoSQL
  - injection de code
---

# Injection

L'injection, c'est lorsque des données non fiables sont liées à quelque chose que le serveur **exécute, analyse, stocke, rend ou requête**, une requête de base de données, une commande du système d'exploitation, du balisage, un runtime de template, un message HTTP, ou une URL sortante. Lorsque les données franchissent du plan *données* vers le plan *code*, l'attaquant contrôle une partie de ce que fait l'interpréteur.

Cet arbre est organisé strictement par la **primitive attaquée**, non par la façon dont l'entrée arrive (query string, en-tête ou corps, cet axe relève de la livraison d'entrée). Chaque enfant nomme la primitive : requêtes de base de données, commandes OS, parseurs de balisage, moteurs de templates, gestion HTTP, requêtes sortantes, et plus.

## Sous-sujets

- **[API](api/index.md)** : le code serveur qui analyse et dispatche des requêtes API structurées décide ce qui s'exécute, pour qui, et contre quel backend.
- **[Command](command/index.md)** : le code applicatif construit une commande du système d'exploitation à partir d'entrée non fiable, laissant un attaquant changer quel programme s'exécute ou avec quels arguments, généralement à distan...
- **[Database](database/index.md)** : de l'entrée non fiable altérant la requête qu'un serveur envoie à son magasin de données, SQL, NoSQL, échappatoires d'ORM, et services d'annuaire.
- **[Expression evaluation](expression-evaluation/index.md)** : lorsqu'une application évalue au runtime une chaîne d'expression influencée par l'attaquant, le langage d'expression devient un chemin de code.
- **[File](file/index.md)** : abus de la façon dont une application web reçoit, résout, inclut ou exporte des fichiers, transformant la manipulation de fichiers en exécution de code, divulgation, ou interpréteur en aval...
- **[HTTP](http/index.md)** : exploiter le message HTTP lui-même : abus des valeurs d'en-tête, response splitting CRLF, request smuggling quand front et back ends divergent, et parameter poll...
- **[Log](log/index.md)** : de l'entrée non fiable écrite dans des lignes ou des champs de log, laissant un attaquant forger des événements, décaler des champs, et confondre les parseurs, agrégateurs et pipelines SIEM qui...
- **[Mail](mail/index.md)** : abus d'email au niveau applicatif où de l'entrée non fiable atteint les en-têtes du message, le corps, ou le hand-off SMTP et est interprétée comme partie du message plutôt...
- **[Markup](markup/index.md)** : injection dans le balisage côté serveur et les langages de traitement XML : Server-Side Includes, analyse XML, requêtes XPath/XQuery/XSLT, et abus d'entités XXE.
- **[Realtime](realtime/index.md)** : canaux longue durée où les corps de messages et les métadonnées pilotent le routage, l'autorisation et les sinks, de sorte que les frames individuelles portent l'injection au-delà de la frontière de la requête HTTP...
- **[Request forgery](request-forgery/index.md)** : lorsque de l'entrée non fiable oriente une requête sortante, un attaquant atteint des services internes, des métadonnées cloud, et des schémas d'URL alternatifs.
- **[Template Engine](template-engine/index.md)** : exploiter l'injection de template côté serveur : détecter le moteur, escalader d'une expression de template vers le runtime du langage hôte, et les limites du sandbox qui...
