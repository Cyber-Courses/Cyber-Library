---
order: 6
title: "Reverse proxy et edge : désaccords edge/origin, exposition du backend et confiance dans les en-têtes"
description: "Attaques de plateforme qui relèvent du rôle de proxy plutôt que d'un produit précis : désaccord de normalisation de chemin entre l'edge et l'origin, accès direct à l'origin et confiance dans les en-têtes injectés par l'edge."
keywords:
  - reverse proxy
  - confusion de chemin
  - exposition de l'origin
  - x-forwarded
  - contournement de WAF
---

# Reverse proxy et edge

Un reverse proxy ou un CDN/WAF placé devant l'origin ajoute un second analyseur de requêtes et une table de routage, de sorte que la sécurité dépend désormais de l'accord entre le proxy et l'origin sur *ce qu'est la requête* et *où elle va*. Ces problèmes relèvent du **rôle** de proxy plutôt que d'un produit serveur précis, ce qui explique qu'ils figurent ici et non sous Apache/nginx/IIS (un bug de proxy propre à un produit, comme la variable `proxy_pass` de nginx, est traité sous son produit).

## Ce qu'il faut vérifier

- **Désaccord de normalisation** : l'edge et l'origin canonicalisent les chemins différemment, de sorte qu'un chemin forgé passe une règle d'autorisation/refus de l'edge mais se résout en une ressource protégée au niveau de l'origin.
- **Exposition de l'origin** : le backend est joignable directement, ce qui contourne entièrement le filtrage, les limites de débit et le contrôle d'accès du proxy.
- **Confiance dans les en-têtes de l'edge** : le proxy injecte ou transmet des en-têtes `X-Forwarded-*`/`Host` que l'origin considère ensuite comme faisant autorité.

## Sujets liés, traités ailleurs

- Le **HTTP request smuggling** (désynchronisation front/back sur les limites de requête) est traité dans [Code > Injection > HTTP](../../code/injection/http/request-smuggling.md).
- Les conséquences côté application de la confiance accordée aux en-têtes transmis sont traitées dans [Code > Access Control > confiance dans les en-têtes de proxy](../../code/access-control/trust-boundary/proxy-header-trust-abuse.md) et [Identification](../../code/identification/rate-limits-and-automation.md) ; ici, l'accent porte sur le câblage de l'edge qui crée cette confiance.

## Pages

- **[Désaccord de normalisation](normalization-mismatch.md)** : l'edge et l'origin ne s'accordent pas sur le chemin canonique, ce qui contourne le contrôle d'accès basé sur le chemin.
- **[Exposition de l'origin](origin-exposure.md)** : atteindre le backend directement, au-delà du proxy, du WAF ou du CDN.
- **[Confiance dans les en-têtes de l'edge](edge-header-trust.md)** : comment le câblage du proxy rend des en-têtes `X-Forwarded-*`/`Host` forgés faisant autorité.

## Références

- PortSwigger Web Security Academy: Access control, SSRF, Web cache
- Orange Tsai: reverse-proxy path confusion research
