---
title: "Firewall rules: opening ingress with compute.firewalls"
order: 1
description: "Exposing or reaching workloads by editing VPC firewall rules with compute.firewalls.create or update to open ingress."
keywords:
  - firewall rules
  - compute.firewalls
  - ingress
  - VPC
  - network exposure
  - 0.0.0.0/0
---

# Firewall rules

GCP firewall rules are VPC-level allow and deny rules matched by direction, source range, target tags or service accounts, and protocol/port. With `compute.firewalls.create` or `compute.firewalls.update` you open a path to an instance you want to reach, and with read access you find the ingress the project already leaves open to the internet.

## Finding wide-open ingress

```bash
# every rule, with direction, source ranges, and allowed ports
gcloud compute firewall-rules list \
  --format="table(name,direction,sourceRanges.list(),allowed[].map().firewall_rule().list())"

# rules that allow the whole internet
gcloud compute firewall-rules list \
  --filter="sourceRanges:0.0.0.0/0 AND direction=INGRESS" \
  --format="table(name,allowed[].map().firewall_rule().list(),targetTags.list())"
```

The default `default-allow-ssh` and `default-allow-rdp` rules on the auto-mode `default` network put `22` and `3389` on the internet for every tagged instance.

## Opening a path you control

```bash
# allow yourself to a target by tag or to the whole VPC
gcloud compute firewall-rules create allow-me \
  --network <vpc> --direction INGRESS --action ALLOW \
  --rules tcp:22,tcp:3389,tcp:3389 --source-ranges <your-ip>/32 \
  --target-tags <tag>
```

## Exploitation notes

- Rules target instances by **network tag** or **service account**; a rule with a broad target tag that many instances carry exposes all of them at once.
- A high-priority `deny` can be shadowed by creating a lower-numbered (higher-priority) `allow`, since lower priority value wins.
- Pair exposed ingress with [Compute Engine](../compute/compute-engine/index.md) to confirm which instances the rule fronts and reach their metadata.

## Tools

- **gcloud** (`compute firewall-rules list`/`create`): enumeration and rule edits.
- **nmap** / **masscan**: confirm reachability of exposed ports from outside.

## References

- [HackTricks Cloud: GCP firewall and network](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google Cloud: VPC firewall rules](https://cloud.google.com/firewall/docs/firewalls)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
