---
title: "Org policy : assouplir les contraintes pour débloquer d'autres escalades"
order: 2
description: "Désactiver des contraintes d'organization policy avec orgpolicy.policy.set pour débloquer la création de clés de compte de service, le partage externe et d'autres escalades bloquées."
keywords:
  - org policy
  - orgpolicy.policy.set
  - constraint
  - organization
  - policy bypass
---

# Org policy

Les contraintes d'organization policy bloquent par défaut plusieurs primitives d'escalade : création de clés de compte de service, utilisation inter-projets de comptes de service, membres IAM externes et adresses IP externes de VM. Avec `orgpolicy.policy.set` au niveau d'un projet, d'un dossier ou d'une organisation, vous désactivez la contrainte qui vous gêne, puis exécutez l'escalade qu'elle empêchait.

## Assouplir une contrainte

```bash
# allow service-account key creation (then use Key creation for durable access)
gcloud resource-manager org-policies disable-enforce \
  constraints/iam.disableServiceAccountKeyCreation --project=<project>

# allow adding members outside the org domain
gcloud resource-manager org-policies delete \
  constraints/iam.allowedPolicyMemberDomains --project=<project>
```

Une fois `iam.disableServiceAccountKeyCreation` désactivée, suivez la [création de clé](../service-account-impersonation/key-creation.md) ; une fois la restriction de domaine levée, une liaison [setIamPolicy](set-iam-policy.md) vers un principal attaquant externe est acceptée.

## Notes d'exploitation

- L'org policy est une barrière, pas une attribution : l'assouplir débloque un autre chemin mais ne donne pas d'accès en soi, c'est donc toujours la première étape d'une chaîne.
- Définir la policy au niveau du projet remplace l'héritage pour ce projet uniquement, ce qui est plus discret que de modifier la valeur par défaut de l'organisation.
- Le changement est journalisé et visible dans la policy : enchaînez donc rapidement avec l'escalade suivante.

## Outils

- **gcloud** (`resource-manager org-policies`) : désactiver ou supprimer des contraintes.

## Références

- [Rhino Security Labs : escalade de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : contraintes d'organization policy](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints)
