---
title: "Front Door et CDN : exposition de l'origine et abus de routage"
order: 5
description: "Abuser d'Azure Front Door et CDN : exposition de l'origine, abus du host-header et du routage, et endpoints orphelins."
keywords:
  - Front Door
  - CDN
  - origine
  - host header
  - routage
---

# Front Door et CDN

Azure Front Door et CDN se placent devant les origines applicatives. Les faiblesses récurrentes sont une **origine** qui accepte le trafic directement (contournant le WAF et les règles de routage en périphérie), une gestion du **host-header** qui permet de router ou de mettre en cache une requête pour un domaine qu'elle ne devrait pas, et des **endpoints orphelins** laissés quand un profil est supprimé mais pas son enregistrement DNS.

## Énumérer les profils et origines

```bash
# Front Door profiles, their endpoints and origin groups
az afd profile list -o table
az afd origin-group list --profile-name <p> -g <rg> -o table
az afd origin list --profile-name <p> -g <rg> --origin-group-name <og> \
  --query "[].{Host:hostName,Origin:name}" -o table
# classic CDN endpoints
az cdn endpoint list --profile-name <p> -g <rg> --query "[].{Name:name,Origin:origins[0].hostName}" -o table
```

## Notes d'exploitation

- Si l'hôte d'origine (un App Service, du stockage ou une VM) est joignable directement, les requêtes qui lui sont envoyées contournent le WAF de Front Door et tout routage par chemin ou en-tête, donc les contrôles en périphérie sont inopérants ; trouvez l'origine et attaquez-la directement.
- Des règles de host-header ou de cache qui s'indexent sur une valeur contrôlable par l'attaquant peuvent empoisonner le cache ou router vers le mauvais backend.
- Un endpoint Front Door ou CDN supprimé dont le nom `*.azureedge.net` / `*.azurefd.net` conserve un CNAME est un [DNS takeover](dns-takeover.md).

## Outils

- **az afd** / **az cdn** : énumération des profils, endpoints et origines.
- **Burp Suite** : test du host-header et de la clé de cache contre la périphérie et l'origine.

## Références

- [HackTricks Cloud : services Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : Azure Front Door](https://learn.microsoft.com/azure/frontdoor/front-door-overview)
- [Can I take over XYZ (empreintes d'endpoints en périphérie)](https://github.com/EdOverflow/can-i-take-over-xyz)
