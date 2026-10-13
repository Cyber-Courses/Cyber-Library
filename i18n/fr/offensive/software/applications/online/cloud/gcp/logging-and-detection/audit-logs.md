---
title: "Journaux d'audit : désactiver la journalisation d'accès aux données"
order: 2
description: "Désactiver ou restreindre les Cloud Audit Logs, en particulier les journaux d'accès aux données, pour masquer les appels d'API."
keywords:
  - Cloud Audit Logs
  - journaux d'accès aux données
  - configuration d'audit
  - journalisation
  - évasion de détection
  - anti-forensique
---

# Journaux d'audit

Les Cloud Audit Logs comportent trois flux : **Admin Activity** (toujours actif, ne peut pas être désactivé), **Data Access** (désactivé par défaut sauf pour quelques services, et celui qui enregistre les lectures et les appels du plan de données) et **System Event**. La configuration d'accès aux données réside dans le bloc `auditConfigs` de la politique IAM, de sorte qu'un principal capable de définir la politique IAM peut désactiver la journalisation d'accès aux données et masquer la majeure partie de l'activité de lecture.

## Lire la configuration d'audit actuelle

```bash
gcloud projects get-iam-policy <project> --format=json > policy.json
# the "auditConfigs" block lists which services log DATA_READ / DATA_WRITE
```

## Désactiver la journalisation d'accès aux données

```bash
# remove or empty the auditConfigs block in policy.json, then write it back
gcloud projects set-iam-policy <project> policy.json
```

Définir `auditConfigs` à vide (ou retirer `DATA_READ`/`DATA_WRITE` pour les services que vous allez toucher) empêche l'enregistrement de ces lectures, tandis qu'Admin Activity continue de journaliser les écritures du plan de contrôle.

## Notes d'exploitation

- **Les journaux Admin Activity ne peuvent pas être désactivés**, donc l'appel `set-iam-policy` qui désactive la journalisation d'accès aux données est lui-même enregistré ; c'est un échange d'une écriture bruyante contre le silence sur toutes les lectures ultérieures.
- La configuration d'audit peut être définie au niveau de l'**organisation, du dossier ou du projet** ; une activation au niveau de l'organisation prime sur une tentative au niveau du projet, vérifiez donc la hiérarchie des ressources avant de supposer que les lectures passent inaperçues.
- Les journaux d'accès aux données sont déjà désactivés par défaut dans la plupart des projets, donc dans de nombreuses cibles, l'activité de lecture est non journalisée sans aucune action.

## Outils

- **gcloud** (`projects get-iam-policy` / `set-iam-policy`) : lire et réécrire la configuration d'audit.
- **ScoutSuite** / **Prowler** : signalent quels services ont la journalisation d'accès aux données activée.

## Références

- [HackTricks Cloud : GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : configurer les journaux d'audit Data Access](https://cloud.google.com/logging/docs/audit/configure-data-access)
