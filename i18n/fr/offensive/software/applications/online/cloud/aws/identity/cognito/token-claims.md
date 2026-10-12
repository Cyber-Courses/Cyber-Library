---
title: "Token claims : abuser des revendications des jetons d'ID et d'accès Cognito"
order: 3
description: "Altérer ou abuser des revendications des jetons d'ID et d'accès Cognito pour assumer une identité plus privilégiée."
keywords:
  - Cognito
  - revendications de jeton
  - jeton d'ID
  - jeton d'accès
  - JWT
---

# Token claims

Cognito émet des jetons JWT d'ID et d'accès dont les revendications (groupes, attributs `custom:`, scopes) sont consommées pour l'autorisation par les applications et les identity pools. Lorsqu'une application fait confiance à une revendication que l'utilisateur peut influencer, ou valide faiblement les jetons, manipuler les revendications élève la session.

## Où les revendications sont dignes de confiance

- **Mappage de rôle d'identity pool** : un pool peut mapper les utilisateurs à des rôles selon une revendication de jeton (un groupe ou un attribut), de sorte que contrôler cette revendication sélectionne un rôle plus privilégié lors de `get-credentials-for-identity`.
- **Autorisation côté application** : une application qui lit `cognito:groups` ou une revendication `custom:` et lui fait confiance accorde tout ce que la revendication indique.

```bash
# decode a held token to see the claims in play
echo "<jwt>" | cut -d. -f2 | base64 -d 2>/dev/null
```

## Notes d'exploitation

- L'appartenance à un groupe pilote `cognito:groups` ; si vous pouvez rejoindre un groupe (défini par un administrateur ou en libre-service), le jeton suivant le porte.
- Les règles de mappage de rôle qui reposent sur un attribut modifiable se combinent directement avec l'abus d'attributs de [user pool](user-pool.md).
- Une validation faible de la signature ou de l'audience côté application laisse passer un jeton forgé ; testez si l'application vérifie le JWKS Cognito et `aud`.

## Outils

- **jwt_tool** : inspecter et tester les revendications du jeton et leur validation.
- **AWS CLI** (`cognito-identity get-credentials-for-identity`) pour transformer un jeton mappé en identifiants de rôle.

## Références

- [HackTricks Cloud: AWS Cognito](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [AWS: Cognito identity-pool role mapping](https://docs.aws.amazon.com/cognito/latest/developerguide/role-based-access-control.html)
- [cognito-scanner](https://pypi.org/project/cognito-scanner/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
