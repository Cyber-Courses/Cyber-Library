---
title: "Confused deputy : assumer des rôles de fournisseur sans l'external ID"
order: 2
description: "Abuser d'un rôle dont la politique de confiance fait confiance à un tiers sans external ID, en profitant de l'accès du deputy pour entrer dans le compte."
keywords:
  - confused deputy
  - external ID
  - trust policy
  - third party
  - AssumeRole
---

# Confused deputy

Les fournisseurs SaaS tiers se voient attribuer un rôle dont la politique de confiance nomme le compte AWS du fournisseur comme principal. AWS a conçu la condition **external ID** pour restreindre cette confiance à votre tenant spécifique ; lorsqu'elle est absente, tout client de ce fournisseur (ou quiconque peut faire assumer le rôle au fournisseur) profite de l'accès du fournisseur pour entrer dans le compte. C'est le problème du confused deputy.

## Repérer la faille

```bash
aws iam get-role --role-name <vendor-role> --query 'Role.AssumeRolePolicyDocument'
# Principal.AWS = the vendor account, and NO Condition on sts:ExternalId
```

Une confiance qui nomme un compte de fournisseur sans `StringEquals` sur `sts:ExternalId` est assumable par ce fournisseur pour le compte de n'importe quel tenant, y compris un que l'attaquant contrôle dans le produit du fournisseur.

## Notes d'exploitation

- L'attaque passe par la plateforme du fournisseur : vous configurez le fournisseur pour qu'il pointe vers le rôle victime, et le fournisseur (le deputy) l'assume pour vous.
- Même avec un external ID, une valeur devinable ou fuitée rouvre la voie : les external ID faibles valent donc autant qu'aucun.
- Croisez avec [cross-account](cross-account.md) pour le cas plus simple où la confiance nomme directement un compte entier.

## Outils

- **AWS CLI** (`iam get-role`) : lire le document de confiance.
- **PMapper** : signale les rôles qui font confiance à des comptes externes.

## Références

- [AWS : le problème du confused deputy](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)
- [HackTricks Cloud : cross-account et confused deputy](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
