---
title: "Network Firewall: finding rule gaps and disabling inspection"
description: "Enumerating and abusing AWS Network Firewall rule groups and policies to find gaps or disable inspection."
keywords:
  - Network Firewall
  - rule group
  - firewall policy
  - inspection
  - bypass
---

# Network Firewall

AWS Network Firewall inspects VPC traffic against stateless and stateful rule groups bound to a firewall policy. Reading those rules shows exactly what is filtered and what is not, so you can route exfiltration or command-and-control through an allowed destination; write access lets you weaken the policy or drop inspection entirely.

## Enumerating rules and policy

```bash
aws network-firewall list-firewalls
aws network-firewall describe-firewall --firewall-name <name>
aws network-firewall describe-firewall-policy --firewall-policy-name <policy>
aws network-firewall describe-rule-group --rule-group-name <rg> --type STATEFUL
```

## Exploitation notes

- The default action for uninspected traffic and any allow-listed FQDNs or CIDRs are the gaps: send exfiltration to an allowed destination rather than fighting the rules.
- A stateful rule group evaluated in the wrong order, or a missing TLS SNI rule, lets traffic through that the operator believes is blocked.
- With write access, disassociating a rule group or setting the policy default to allow removes inspection without deleting the firewall, which looks benign in an inventory.

## Tools

- **AWS CLI** (`network-firewall describe-*`): rule-group and policy disclosure.
- **ScoutSuite** / **Prowler**: inventory firewalls and default actions.

## References

- [AWS: Network Firewall rule groups](https://docs.aws.amazon.com/network-firewall/latest/developerguide/rule-groups.html)
- [HackTricks Cloud: AWS Network Firewall](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
