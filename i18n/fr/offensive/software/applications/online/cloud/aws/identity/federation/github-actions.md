---
title: "GitHub Actions : assumer des rôles via un subject OIDC trop permissif"
order: 3
description: "Abuser d'une confiance OIDC GitHub Actions trop peu restreinte (revendication subject en wildcard) pour assumer un rôle depuis un workflow contrôlé par l'attaquant."
keywords:
  - GitHub Actions
  - OIDC
  - revendication sub
  - workflow
  - AssumeRoleWithWebIdentity
---

# GitHub Actions

GitHub Actions peut s'authentifier auprès d'AWS via le fournisseur OIDC de GitHub, en assumant un rôle avec `sts:AssumeRoleWithWebIdentity` et sans secret stocké. La stratégie de confiance du rôle est censée épingler la revendication `sub` du jeton à un `repo:org/name:...` spécifique. Lorsque cette condition est en wildcard ou limitée uniquement à l'organisation, un workflow dans n'importe quel dépôt correspondant, y compris un fork ou un dépôt que vous créez, assume le rôle.

## La confiance trop permissive

```bash
aws iam get-role --role-name <role> --query 'Role.AssumeRolePolicyDocument'
# look for token.actions.githubusercontent.com:sub with StringLike "repo:org/*"
# or a condition on :aud only, with no :sub pin
```

Un `sub` comme `repo:org/*:ref:refs/heads/*` accepte n'importe quelle branche de n'importe quel dépôt de l'organisation ; un `sub` absent accepte n'importe quel dépôt pour lequel GitHub émettra un jeton.

## Assumer depuis un workflow

Un workflow que vous contrôlez dans un dépôt correspondant demande le jeton OIDC et assume :

```yaml
permissions: { id-token: write }
steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::<acct>:role/<role>
      aws-region: us-east-1
```

## Notes d'exploitation

- La restriction limitée à la seule organisation est l'erreur courante : une organisation publique laisse des tiers ouvrir un dépôt qui satisfait `repo:org/*`.
- La revendication `aud` seule n'est pas une restriction ; sans condition `sub`, tout jeton émis par le fournisseur est accepté.
- Voir [OIDC](oidc.md) pour la mécanique générale de web-identity derrière cela.

## Outils

- **aws-actions/configure-aws-credentials** : l'action d'assumption standard.
- **AWS CLI** (`sts assume-role-with-web-identity`) avec un jeton obtenu depuis le runtime Actions.

## Références

- [GitHub: configuring OpenID Connect in AWS](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [Rhino Security Labs: AWS privilege escalation](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
