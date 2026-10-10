---
title: "DNS takeover : revendiquer les enregistrements DNS Azure orphelins"
order: 4
description: "Revendiquer les enregistrements DNS Azure orphelins laissés par des ressources App Service, Traffic Manager, CDN ou IP publiques supprimées."
keywords:
  - DNS takeover
  - DNS orphelin
  - App Service
  - Traffic Manager
  - subdomain takeover
---

# DNS takeover

Un enregistrement orphelin pointe un nom vers une ressource Azure qui n'existe plus : un App Service supprimé (`*.azurewebsites.net`), un profil Traffic Manager retiré (`*.trafficmanager.net`), un endpoint CDN ou Front Door démantelé (`*.azureedge.net`), un site statique de stockage supprimé, ou une IP publique libérée. Comme le nom résout toujours vers l'espace de noms Azure, quiconque recrée une ressource avec le même identifiant capture chaque requête vers ce sous-domaine, un point d'ancrage à origine de confiance pour le phishing, le vol de cookies et l'émission de certificats.

## Trouver les enregistrements orphelins

```bash
# records whose target is an Azure namespace, then resolve and flag NXDOMAIN / unclaimed
az network dns zone list -o table
az network dns record-set list -g <rg> -z <zone> \
  --query "[?cnameRecord].{Name:name,Target:cnameRecord.cname}" -o table
```

## Revendiquer la ressource sous-jacente

- **App Service** : un CNAME vers `victim.azurewebsites.net` renvoyant NXDOMAIN signifie que vous créez une Web App nommée `victim` dans la même région publique et servez du contenu.
- **Traffic Manager / CDN / Front Door** : recréez un profil ou un endpoint avec le même label (`victim.trafficmanager.net`, `victim.azureedge.net`) sur une ressource que vous possédez.
- **Site statique de stockage / IP publique** : recréez le nom du compte de stockage, ou réallouez des IP publiques jusqu'à obtenir l'adresse référencée.

## Notes d'exploitation

- Un sous-domaine capturé hérite de la confiance du domaine parent : les cookies cadrés sur `.example.com`, les listes d'autorisation de redirection SSO et l'attente des utilisateurs se transfèrent.
- Le contrôle du nom valide les défis ACME HTTP-01 et DNS-01, donc vous pouvez émettre un certificat valide pour le sous-domaine.
- Les propres recommandations de Microsoft notent que des milliers de sous-domaines sont vulnérables à travers les tenants à tout moment, car les suppressions nettoient rarement l'enregistrement DNS.

## Outils

- **empreintes de subdomain-takeover** : [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) liste les services Azure et leurs signatures revendicables.
- **nuclei** (templates `takeovers`), **subjack**, **tko-subs** : détectent les enregistrements revendicables à grande échelle.
- **az cli** : recréer l'App Service, le profil Traffic Manager ou le compte de stockage pour le revendiquer.

## Références

- [HackTricks Cloud : DNS takeover Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Can I take over XYZ (empreintes de takeover)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Microsoft : prévenir les entrées DNS orphelines](https://learn.microsoft.com/azure/security/fundamentals/subdomain-takeover)
