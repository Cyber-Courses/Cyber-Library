---
order: 3
title: "Abus de la méthode HTTP et de la ligne de requête"
description: "Abuser de la ligne de requête HTTP : en-têtes d'override de méthode et verb tampering pour contourner le contrôle d'accès basé sur la méthode, et injection dans la ligne de requête."
keywords:
  - override de méthode
  - X-HTTP-Method-Override
  - verb tampering
  - injection de ligne de requête
  - méthode HTTP
---

# Ligne de requête

La ligne de requête (`METHOD URI VERSION`) détermine comment une requête est routée et autorisée, et les applications qui prennent des décisions d'accès sur la méthode peuvent être subverties en changeant quelle méthode est appliquée.

Les mécanismes d'override de méthode permettent à un client de changer la méthode effective après que le front-end a vu l'originale. De nombreux frameworks honorent `X-HTTP-Method-Override`, `X-HTTP-Method`, `X-Method-Override`, ou un champ `_method` de corps/query, convertissant un `POST` en `PUT`/`DELETE`/`PATCH`. Là où un proxy ou un WAF applique des contrôles par méthode (bloquant `DELETE` mais autorisant `POST`), l'override atteint le back end en tant que méthode privilégiée et contourne ce contrôle :

```
POST /api/users/5 HTTP/1.1
X-HTTP-Method-Override: DELETE
```

Le verb tampering abuse d'une gestion incomplète des méthodes. Si une règle d'accès est écrite pour des verbes spécifiques (`GET` et `POST`), une méthode peu courante telle que `HEAD`, `PUT`, ou un token arbitraire peut passer devant la règle tandis que l'application traite tout de même la requête, une mauvaise configuration classique dans les filtres d'autorisation de conteneurs et de frameworks.

L'injection dans la ligne de requête proprement dite survient lorsqu'une application construit une ligne de requête sortante à partir d'entrée (un client HTTP ou un proxy construisant `GET <input> HTTP/1.1`). Comme le token de méthode précède l'entrée, les espaces ou CR/LF non échappés ne peuvent pas changer la méthode de cette requête, mais ils peuvent altérer le chemin cible ou la version, ajouter des en-têtes, ou, avec un cadrage de requête complet, injecter une requête ultérieure séparée dont la méthode diffère. Ceci chevauche la falsification de requête côté serveur lorsque la requête influencée atteint un service interne, donc c'est souvent chaîné là.

Le test consiste à renvoyer une requête avec chaque en-tête d'override et chaque verbe inhabituel et à guetter un changement de résultat d'autorisation ou de routage.

## Outils

- **Burp Repeater** : permuter les méthodes, ajouter des en-têtes d'override, et surveiller les changements d'autorisation et de routage.
- **Turbo Intruder** : itérer à grande échelle sur les en-têtes d'override et les verbes inhabituels.
- **curl** : envoyer des méthodes arbitraires et des en-têtes d'override de méthode depuis la ligne de commande.

## Références

- OWASP Testing Guide: Testing for HTTP Verb Tampering
- RFC 9112: HTTP/1.1 request line
