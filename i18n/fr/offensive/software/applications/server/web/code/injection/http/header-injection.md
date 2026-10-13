---
order: 1
title: "Injection d'en-tête HTTP : en-têtes Host et de forwarding"
description: "Abuser des en-têtes HTTP contrôlés par l'attaquant auxquels l'application fait confiance : l'en-tête Host pour l'empoisonnement de la réinitialisation de mot de passe et du cache, et les en-têtes X-Forwarded / d'override pour le contournement du contrôle d'accès et le SSRF."
keywords:
  - injection d'en-tête host
  - empoisonnement de réinitialisation de mot de passe
  - X-Forwarded-For
  - X-Forwarded-Host
  - X-Original-URL
---

# Injection d'en-tête

Les applications font souvent confiance à des en-têtes de requête que le client contrôle entièrement. Lorsqu'une valeur d'en-tête alimente le routage, le contrôle d'accès ou la génération de liens, la régler sur une valeur de l'attaquant subvertit cette logique. Aucun CRLF n'est nécessaire ici ; l'abus réside dans la valeur elle-même (le response splitting CRLF est couvert séparément).

L'en-tête `Host` est la cible classique, car les applications le réutilisent pour construire des URL absolues. Un flux de réinitialisation de mot de passe qui construit le lien de réinitialisation à partir de `Host` envoie à la victime un lien pointant vers le domaine de l'attaquant, fuitant le token lorsqu'il est cliqué :

```
POST /reset HTTP/1.1
Host: attacker.tld
...
email=victim@example.com
```

Le même `Host` reflété permet l'empoisonnement de cache web (une page mise en cache avec une URL de ressource absolue contrôlée par l'attaquant) et, là où le routage de virtual-host ou les règles d'accès se fondent sur `Host`, le contournement du routage et des ACL. Lorsque le front end fixe `Host`, les overrides `X-Forwarded-Host` et `X-Host` atteignent souvent l'application et sont honorés à la place.

Les en-têtes de forwarding et d'override portent une confiance que l'attaquant peut forger. `X-Forwarded-For` est jugé fiable pour le contrôle d'accès basé sur l'IP et le rate limiting, donc l'usurper contourne les listes d'autorisation d'IP ou réinitialise les compteurs. `X-Forwarded-Host`/`X-Forwarded-Proto` influencent la génération de liens et de redirections. `X-Original-URL` et `X-Rewrite-URL` font router certaines stacks vers un chemin différent de celui que le contrôle d'accès du front-end a vérifié, contournant les restrictions basées sur le chemin (requêter `/` avec `X-Original-URL: /admin`).

Le test consiste à envoyer chaque en-tête avec une valeur de l'attaquant et à guetter sa présence dans un lien, une décision de routage, ou un résultat d'autorisation. La portée dépend du framework et de la chaîne de proxy, donc confirmez quels en-têtes le back end honore réellement.

## Outils

- **Burp Suite / Burp Repeater** : injecter les en-têtes `Host` et de forwarding/override et surveiller les liens, le routage et les résultats d'authentification.
- **curl** : envoyer des en-têtes `Host`, `X-Forwarded-*` et `X-Original-URL` arbitraires depuis la ligne de commande.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- PortSwigger Web Security Academy: HTTP Host header attacks
- OWASP Testing Guide: Testing for Host Header Injection
