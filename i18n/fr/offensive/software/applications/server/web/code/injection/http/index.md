---
order: 4
title: "Injection HTTP : en-têtes, CRLF, smuggling et parameter pollution"
description: "Exploiter le message HTTP lui-même : abus des valeurs d'en-tête, response splitting CRLF, request smuggling quand front et back ends divergent, et HTTP parameter pollution."
keywords:
  - injection HTTP
  - injection CRLF
  - request smuggling
  - injection d'en-tête host
  - HTTP parameter pollution
---

# HTTP

L'injection HTTP traite le message HTTP comme la primitive dangereuse : lorsqu'une application ou un intermédiaire construit ou analyse des requêtes et des réponses à partir d'entrées non fiables, la structure du message elle-même peut être subvertie. Ceci se distingue de l'injection dans une requête ou une commande portée à l'intérieur de la requête ; ici la cible est les en-têtes, la structure des lignes, le cadrage du corps, et la façon dont une chaîne de proxys et de serveurs s'accorde (ou non) sur l'endroit où un message se termine.

Les attaques se répartissent en quelques familles. L'abus des valeurs d'en-tête fait confiance à des en-têtes posés par l'attaquant tels que `Host` et `X-Forwarded-*` pour le routage, le contrôle d'accès ou la génération de liens. L'injection CRLF scinde une réponse lorsqu'un retour chariot/saut de ligne atteint un en-tête de réponse. Le request smuggling désynchronise un front-end et un back-end qui mesurent la longueur de requête différemment. La parameter pollution fournit des paramètres dupliqués que les composants résolvent de façon incohérente.

Un thème récurrent est que les serveurs et frameworks modernes ont durci beaucoup de ces techniques (retrait des CR/LF des API d'en-têtes, normalisation de `Content-Length`/`Transfer-Encoding`), donc la portée de chaque technique dépend de la stack exacte, et le fingerprinting du front-end et du back-end fait partie du travail.

## Techniques

- **[Header injection](header-injection.md)** : abuser des en-têtes `Host` et de forwarding/override pour l'empoisonnement et le contournement d'accès.
- **[CRLF and response splitting](crlf-and-response-splitting.md)** : injecter des CR/LF dans un en-tête de réponse pour ajouter des en-têtes ou scinder la réponse.
- **[Request smuggling](request-smuggling.md)** : désynchroniser le front-end et le back-end avec `CL.TE`, `TE.CL` et `TE.TE`.
- **[Parameter pollution](parameter-pollution.md)** : dupliquer des paramètres que les composants résolvent différemment.
- **[Request line](request-line.md)** : override de méthode et abus de la ligne de requête.
- **[Request body](request-body.md)** : confusion de content-type et d'analyse du corps.

## Outils

- **Burp Suite** : plateforme centrale pour forger et rejouer des messages HTTP malformés à travers ces techniques.
- **Burp HTTP Request Smuggler** : extension Burp pour détecter et exploiter le désync de requêtes.
- **curl** : envoyer des en-têtes bruts, des CRLF et des content types alternatifs depuis la ligne de commande.

## Références

- RFC 9110/9112 (sémantique HTTP et messagerie HTTP/1.1)
- PortSwigger Web Security Academy: HTTP request smuggling, Host header attacks
