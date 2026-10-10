---
title: "SSRF vers IMDS : voler un jeton d'identité managée via une application web"
description: "Atteindre le point de terminaison IMDS Azure via une server-side request forgery dans une application pour voler le jeton d'identité managée de l'hôte."
keywords:
  - SSRF
  - IMDS
  - identité managée
  - vol de jeton
  - point de terminaison de métadonnées
---

# SSRF vers IMDS

Quand une application sur une ressource Azure peut être contrainte à récupérer une URL choisie par l'attaquant, pointez-la vers le point de terminaison de jeton de métadonnées et la réponse est le jeton d'identité managée de l'hôte. C'est le pont entre une vulnérabilité web et des identifiants Azure complets, ce qui explique pourquoi la [SSRF](../../../../../server/web/code/injection/request-forgery/index.md) est traitée comme une technique d'obtention d'identifiants cloud.

## La requête centrale

```
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/
```

Le piège propre à Azure : IMDS exige l'en-tête **`Metadata: true`** et rejette les requêtes portant un `X-Forwarded-For`. Le sink SSRF doit donc vous laisser définir un en-tête, ou être un sink qui transmet vos en-têtes. Là où l'application définit ses propres en-têtes que vous ne pouvez pas contrôler, le point de terminaison refuse, ce qui détermine quels sinks sont utilisables.

```
# sinks that let you control headers or the raw request are the usable ones:
#   a proxy/webhook that forwards caller headers, a PDF/SSRF gadget with header injection,
#   or a gopher:// sink that writes the full HTTP request including Metadata: true
```

## Contournements quand l'IP littérale est filtrée

```
http://169.254.169.254/   ->  http://[::ffff:169.254.169.254]/   (IPv6-mapped)
                              http://2852039166/                   (decimal)
# DNS rebinding and open-redirect chains where the fetcher follows 3xx
```

## Notes d'exploitation

- Les mécanismes de la SSRF web (sinks, filtres, contournements par redirection et rebinding) se trouvent dans [request forgery](../../../../../server/web/code/injection/request-forgery/index.md) ; cette page ne concerne que la charge utile Azure et la contrainte `Metadata: true`.
- Générez d'abord le jeton `management.azure.com` pour cartographier le RBAC de l'identité, puis redemandez pour `graph.microsoft.com` ou `vault.azure.net` selon les besoins.
- Une SSRF dans App Service ne peut pas atteindre 169.254.169.254 (pas d'IMDS là-bas) ; ciblez plutôt la valeur d'environnement `IDENTITY_ENDPOINT`, qu'une SSRF qui fait fuir l'environnement ou atteint le port local peut atteindre.

## Outils

- **Burp Suite** ou un client personnalisé pour piloter le sink SSRF avec l'en-tête.
- **az rest** / curl pour rejouer le jeton volé.

## Références

- [HackTricks Cloud : SSRF vers IMDS Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : sécurité IMDS (en-tête Metadata)](https://learn.microsoft.com/azure/virtual-machines/instance-metadata-service)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
