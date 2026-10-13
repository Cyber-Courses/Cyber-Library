---
order: 3
title: "Confiance des en-têtes edge : X-Forwarded et Host forgés acceptés comme faisant autorité"
description: "Exploitation d'un câblage de proxy qui rend les en-têtes X-Forwarded-* et Host fournis par le client faisant autorité à l'origine : usurpation d'IP pour l'accès et les limites de débit, et routage basé sur Host vers des applications internes."
keywords:
  - x-forwarded-for
  - en-tête host
  - confiance des en-têtes edge
  - routage interne
  - usurpation d'ip
---

# Confiance des en-têtes edge

Un reverse proxy est censé *positionner* des métadonnées de transmission dignes de confiance (la vraie IP cliente, le schéma et l'hôte d'origine) et *retirer* toute copie fournie par le client. Quand le câblage est erroné, l'origine fait confiance à des en-têtes que le client contrôle entièrement, si bien qu'un attaquant forge l'identité, l'adresse source et le routage. Cette page traite l'angle du câblage edge ; les conséquences côté application vivent dans [Code > Contrôle d'accès](../../code/access-control/trust-boundary/proxy-header-trust-abuse.md) et [Identification](../../code/identification/rate-limits-and-automation.md).

## X-Forwarded-* forgés pour les décisions basées sur l'IP

Si le proxy ajoute à (plutôt que remplace) `X-Forwarded-For`, ou si l'origine lit la *première* valeur, ou si l'origine est joignable directement, un `X-Forwarded-For` fourni par le client est cru :

```
X-Forwarded-For: 127.0.0.1
X-Forwarded-For: 10.0.0.5, <real>
X-Real-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
```

Usages : atteindre des chemins d'administration filtrés par IP (« autoriser 127.0.0.1/interne uniquement »), vaincre les limites de débit et verrouillages par IP en faisant tourner l'en-tête à chaque requête, et usurper les vérifications géo/liste d'autorisation. La fiabilité dépend entièrement de la chaîne de proxy, testez donc quel en-tête et quelle position (première vs dernière) l'origine honore.

## Schéma et hôte forgés

- `X-Forwarded-Proto`/`X-Forwarded-Host` que l'application utilise pour construire des URL absolues permettent des problèmes de cache et de redirection et alimentent l'[empoisonnement de réinitialisation de mot de passe](../../code/authentication/password-reset-and-invite-tokens.md).
- `X-Forwarded-Port` et similaires influencent la génération de liens et certaines vérifications d'accès.

## Routage basé sur Host vers des applications internes

De nombreux déploiements routent par `Host` au niveau du proxy. Fournir un nom d'hôte interne peut atteindre un backend non prévu ou un vhost par défaut/interne :

```
GET / HTTP/1.1
Host: internal-admin.corp.local

GET / HTTP/1.1
Host: localhost
```

Lorsque le proxy transmet un `Host` inconnu à un backend par défaut, ou sélectionne l'amont à partir du `Host`, cela atteint des applications de préproduction, d'administration ou réservées à l'interne. Combinez avec l'[exposition de l'origine](origin-exposure.md) (vhost par défaut) et le cas [proxy_pass variable](../nginx/variable-proxy-pass-ssrf.md) de nginx où `Host` choisit l'amont.

## Exploitation

- Énumérez quels en-têtes de transmission l'origine honore en envoyant chacun contre un chemin filtré par IP ou un endpoint à limite de débit et en observant l'effet.
- Faites tourner `X-Forwarded-For` à chaque requête pour contourner la limitation par IP pendant le brute force et le credential stuffing.
- Essayez des noms d'hôte internes et `localhost` dans `Host` pour atteindre des vhosts internes et des backends d'administration.

## Outils

- Burp (Repeater/Intruder, rotation d'en-têtes) ; l'extension Param Miner pour la découverte d'en-têtes.

## Références

- PortSwigger Web Security Academy : Host header attacks ; HTTP headers and access control
- nginx/Apache : configuration real_ip / IP distante
