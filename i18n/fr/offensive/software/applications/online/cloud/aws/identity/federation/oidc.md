---
title: "OIDC : AssumeRoleWithWebIdentity et conditions de revendication faibles"
order: 2
description: "Abuser d'un fournisseur d'identité OIDC IAM via sts:AssumeRoleWithWebIdentity lorsque les conditions sub et aud de la stratégie de confiance du rôle sont absentes ou trop larges."
keywords:
  - OIDC
  - AssumeRoleWithWebIdentity
  - web identity
  - revendication sub
  - revendication aud
---

# OIDC

Un fournisseur OIDC IAM fait confiance aux jetons signés par un émetteur externe. Un rôle qui lui est lié est assumé avec `sts:AssumeRoleWithWebIdentity`, et la seule chose qui sépare un attaquant du rôle est le bloc `Condition` de la stratégie de confiance sur les revendications `sub` (subject) et `aud` (audience) du jeton. Si celles-ci sont absentes ou en wildcard, tout jeton que l'émetteur accepte de forger, y compris pour une identité que l'attaquant contrôle, assume le rôle.

## Assumer avec un jeton web-identity

```bash
aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::<acct>:role/<role> \
  --role-session-name s \
  --web-identity-token "$OIDC_JWT" --query Credentials
```

## Lire la condition de confiance

```bash
aws iam get-role --role-name <r> --query 'Role.AssumeRolePolicyDocument'
# look for the OIDC provider as Principal.Federated and the StringEquals/StringLike
# conditions on "<issuer>:sub" and "<issuer>:aud"
```

Une confiance avec `StringLike` et un `*` dans `sub`, ou sans aucune condition `sub`, est assumable par n'importe quel subject servi par l'émetteur.

## Notes d'exploitation

- La vérification d'audience n'est aussi forte que la condition `aud` ; un `aud` absent laisse passer un jeton forgé pour une autre application.
- Pour les émetteurs managés que vous ne contrôlez pas, vous l'emportez quand même lorsque la condition `sub` est assez permissive pour correspondre à une identité que vous pouvez obtenir de cet émetteur.
- GitHub Actions est le cas concret le plus courant et dispose de sa propre [page](github-actions.md).

## Outils

- **AWS CLI** (`sts assume-role-with-web-identity`) : l'assumption.
- **jwt_tool** : inspecter et forger les revendications d'un JWT web-identity.

## Références

- [HackTricks Cloud: OIDC web-identity abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [AWS: AssumeRoleWithWebIdentity](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
