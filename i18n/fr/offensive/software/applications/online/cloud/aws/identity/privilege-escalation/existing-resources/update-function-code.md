---
title: "UpdateFunctionCode : écraser une Lambda qui exécute un rôle privilégié"
description: "lambda:UpdateFunctionCode pour écraser une fonction qui s'exécute déjà avec un rôle d'exécution privilégié."
keywords:
  - UpdateFunctionCode
  - Lambda
  - rôle d'exécution
  - code
  - écrasement
---

# UpdateFunctionCode

Une Lambda existante porte déjà un **rôle d'exécution**. `lambda:UpdateFunctionCode` remplace le code de la fonction, de sorte que vous l'écrasez avec le vôtre et laissez la prochaine invocation s'exécuter en tant que le rôle, sans aucun `PassRole`.

## Écraser et déclencher

```bash
# package a handler that dumps the role creds or does your work
aws lambda update-function-code --function-name <fn> --zip-file fileb://f.zip
# invoke directly, or wait for the function's existing trigger to fire it
aws lambda invoke --function-name <fn> out.json ; cat out.json
```

## Notes d'exploitation

- Il vous faut un accès en écriture sur la fonction mais pas `PassRole`, puisque le rôle est déjà attaché.
- Si vous n'avez pas `InvokeFunction`, une source d'événements existante (API Gateway, S3, planification) exécutera votre code au prochain déclenchement.
- Restaurez le code d'origine ensuite pour limiter la perturbation si la fonction est en usage de production.

## Outils

- **AWS CLI** (`lambda update-function-code` / `invoke`).
- **Pacu** : modules Lambda.

## Références

- [Rhino Security Labs: AWS privilege escalation (UpdateFunctionCode)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
