---
title: "Identité managée : voler le jeton de l'identité de la VM depuis IMDS"
order: 3
description: "Voler le jeton de l'identité managée attachée à une VM depuis IMDS pour agir sur tout l'abonnement."
keywords:
  - identité managée de VM
  - IMDS
  - jeton
  - abonnement
  - escalade de privilèges
---

# Identité managée

Une VM dotée d'une **identité managée** assignée par le système ou par l'utilisateur expose les jetons de cette identité sur le point de terminaison de métadonnées d'instance. Une fois que vous exécutez du code sur la VM (via [run command](run-command.md), une [extension](custom-script-extension.md) ou un shell), vous générez un jeton pour n'importe quelle audience Azure que l'identité peut atteindre et agissez en son nom, ce qui correspond fréquemment à Contributor ou Owner quelque part.

## Générer le jeton sur la machine

```bash
# management plane
curl -s -H Metadata:true \
 "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
# other audiences: https://graph.microsoft.com/ , https://vault.azure.net
# user-assigned: add &client_id=<clientId> or &mi_res_id=<resourceId>
```

Fournissez le jeton bearer retourné à `az rest` ou à l'API ARM et énumérez ce que l'identité peut faire.

## Notes d'exploitation

- C'est la face Azure du schéma du jeton de métadonnées ; les mécanismes de récupération (sélecteurs d'identités assignées par l'utilisateur, `IDENTITY_ENDPOINT` d'App Service, SSRF) se trouvent sous [identifiants](../../credentials/instance-metadata/index.md).
- Attacher une identité privilégiée à une VM que vous contrôlez est en soi une escalade, traitée sous [identité](../../identity/managed-identities/index.md).
- Une identité assignée par l'utilisateur partagée entre de nombreuses ressources élargit le rayon d'impact : un seul jeton de VM agit partout où cette identité est assignée.

## Outils

- **Azure CLI** (`az rest`, `az account get-access-token`) une fois le jeton exporté.
- **MicroBurst** / **ROADtools** : énumération des identités et manipulation des jetons.

## Références

- [Microsoft : identités managées et IMDS](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
- [SpecterOps : chemins d'attaque par identité managée (BARK)](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : identités managées Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
