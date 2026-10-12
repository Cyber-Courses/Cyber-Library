---
title: "Cross-account : assumer des rôles qui font confiance à un autre compte"
order: 1
description: "Abuser des politiques de confiance IAM qui nomment un compte externe entier ou le root, permettant à tout principal que vous détenez là-bas d'assumer dans le compte cible."
keywords:
  - cross-account
  - AssumeRole
  - trust policy
  - root principal
  - lateral movement
---

# Cross-account

La politique de confiance d'un rôle peut nommer un **principal dans un autre compte**. Lorsqu'elle nomme le root du compte (`arn:aws:iam::<acct>:root`) plutôt qu'un rôle spécifique, *tout* principal de ce compte disposant de `sts:AssumeRole` peut assumer le rôle, ce qui transforme un point d'appui dans un compte en accès dans un autre.

## Repérer la confiance

```json
{ "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::111111111111:root" },
  "Action": "sts:AssumeRole" }
```

Ici, `root` désigne le compte entier, et pas seulement son utilisateur root : il délègue la décision d'autorisation à l'IAM du compte `111111111111` lui-même.

## Assumer à travers les comptes

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<target>:role/<role> \
  --role-session-name x --query Credentials
```

## Notes d'exploitation

- Énumérez les documents de confiance à l'échelle du compte avec `get-account-authorization-details`, puis filtrez les valeurs `Principal.AWS` qui sont des roots de compte ou des jokers.
- Le chaînage de rôles fonctionne : assumez le rôle A dans le compte 1, puis utilisez-le pour assumer le rôle B dans le compte 2, en suivant le graphe de confiance que PMapper dessine.
- Une session assumée par chaînage est plafonnée à une heure et ne peut pas être renouvelée par un nouveau chaînage, ce qui compte pour les opérations longues.

## Outils

- **AWS CLI** (`sts assume-role`) : l'assomption.
- **PMapper** : arêtes cross-account lorsque les graphes des deux comptes sont chargés.
- **Pacu** (`iam__enum_assume_role`) : force brute des ARN de rôle candidats pour trouver des rôles assumables.

## Références

- [HackTricks Cloud : abus de confiance cross-account](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
