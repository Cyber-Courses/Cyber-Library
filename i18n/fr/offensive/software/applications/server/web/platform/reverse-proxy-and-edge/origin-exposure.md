---
order: 2
title: "Exposition de l'origin : atteindre le backend directement, au-delà du proxy, du WAF ou du CDN"
description: "Trouver et atteindre le serveur d'origin situé derrière un reverse proxy, un WAF ou un CDN pour contourner le contrôle d'accès, la limitation de débit et le filtrage de l'edge : découverte de l'IP d'origin, confirmation du vhost par défaut et requêtes directes vers l'origin."
keywords:
  - exposition de l'origin
  - contournement de WAF
  - contournement de CDN
  - IP d'origin
  - vhost par défaut
---

# Exposition de l'origin

Lorsque la sécurité n'est appliquée qu'au niveau de l'edge (un WAF, un CDN ou un reverse proxy), atteindre l'**origin directement** contourne tout cela : filtrage, limites de débit, règles géo/IP et contrôles d'accès qui ne vivent que dans l'étage frontal. L'attaque consiste à découvrir l'adresse de l'origin et à lui envoyer les requêtes directement, avec le nom d'hôte public dans l'en-tête `Host` afin que l'application route toujours correctement.

## Trouver l'origin

- **Historique DNS et sous-domaines** : les enregistrements A actuels et historiques (transparence des certificats via `crt.sh`, DNS passif, SecurityTrails) révèlent souvent l'IP antérieure au CDN ou un hôte `origin.`/`direct.`/`staging.` qui se résout directement vers le backend.
- **IP fuitée** : en-têtes d'e-mails émis par l'application (chaîne `Received:` du mail de réinitialisation de mot de passe), pages d'erreur verbeuses, `server-status`, réponses de SSRF et autres services sur le même hôte.
- **Pivots par certificat et favicon** : scanner l'espace d'adressage (Shodan/Censys) à la recherche du certificat TLS ou du hash de favicon de l'application pour repérer l'origin parmi des hôtes sans rapport.
- **Confirmation du vhost par défaut** : une IP qui sert l'application sur son **hôte virtuel par défaut** (pas de `Host`, ou un `Host` sans correspondance) confirme l'origin. Un vhost par défaut sert aussi parfois une application *différente*, moins protégée, ou une page de statut, ce qui constitue en soi une découverte.

## L'atteindre directement

Envoyer la requête vers l'IP d'origin tout en préservant le `Host` public afin que le routage par hôte virtuel et le comportement de l'application restent normaux :

```bash
curl -k https://ORIGIN_IP/admin -H 'Host: app.example.com'
# or pin the hostname to the IP:
curl -k --resolve app.example.com:443:ORIGIN_IP https://app.example.com/admin
```

Si la réponse correspond au vrai site, l'origin est joignable et l'edge est contourné. Essayer aussi un `Host` vide ou incorrect, pour voir ce que sert le vhost par défaut.

## Exploitation

- Rejouer directement sur l'origin, qui n'a aucun filtrage, les payloads bloqués par le WAF (SQLi, XSS, traversal).
- Vaincre la limitation de débit et les règles anti-bot de l'edge pour le credential stuffing et le brute force en ciblant l'origin.
- Atteindre des chemins d'administration ou internes qui n'étaient filtrés par ACL qu'au niveau de l'edge, ce qui recoupe le [désaccord de normalisation](normalization-mismatch.md) (même objectif, route différente).

## Outils

- **crt.sh**, **SecurityTrails**, **Shodan/Censys**, découvreurs d'origin de type CloudFlair ; **curl --resolve**.

## Références

- PortSwigger Web Security Academy: Web application firewalls (bypassing)
- OWASP WSTG: Testing for network/infrastructure configuration
