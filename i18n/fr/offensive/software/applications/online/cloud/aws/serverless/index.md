---
title: "AWS serverless"
order: 5
description: "Attaquer le serverless AWS : rôles d'exécution Lambda, code et couches, politiques de ressource et contournement d'autorisation API Gateway, URL de fonction, Step Functions et EventBridge."
keywords:
  - Lambda
  - rôle d'exécution
  - API Gateway
  - URL de fonction
  - serverless
---

# Serverless

Un composant serverless s'exécute avec un rôle et détient une configuration qui porte fréquemment des secrets, de sorte qu'atteindre l'un revient à atteindre son **rôle** et sa **configuration**. Lambda en est le centre de gravité : une fonction s'exécute sous un **rôle d'exécution**, ses variables d'environnement contiennent souvent des identifiants, et son code est remplaçable par un attaquant. Autour d'elle, API Gateway expose les fonctions et d'autres services AWS, les URL de fonction les exposent directement, et Step Functions ainsi qu'EventBridge orchestrent des actions privilégiées sous leurs propres rôles.

## Ce qui s'intègre ici

- **[Rôle d'exécution](execution-role.md)** : voler les identifiants du rôle d'une fonction Lambda depuis le runtime.
- **[Code et couches](code-and-layers.md)** : lire le code source et les couches à la recherche de secrets, et les écraser pour exécuter du code sous le rôle.
- **[API Gateway](api-gateway/index.md)** : exposition par politique de ressource, contournement d'autorisation et rôle d'intégration.
- **[URL de fonction](function-url.md)** : une URL de fonction Lambda à l'authentification laxiste, invoquée directement via HTTPS.
- **[Step Functions](step-functions.md)** : machines à états et leur rôle d'exécution.
- **[EventBridge](eventbridge.md)** : règles, bus et cibles déclenchées sous un rôle privilégié.

Créer ou mettre à jour une fonction pour lui faire assumer un rôle plus puissant relève du chemin [iam:PassRole](../identity/privilege-escalation/pass-role/lambda.md) et se trouve dans la section identité ; les pages présentes ici couvrent le détournement des services eux-mêmes.

## Références

- [HackTricks Cloud : AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS : rôle d'exécution Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Rhino Security Labs : élévation de privilèges AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
