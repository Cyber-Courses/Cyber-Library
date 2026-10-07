---
title: "Network security groups: exposing and reaching filtered resources"
order: 1
description: "Reading and rewriting Azure network security group rules to expose or reach otherwise filtered resources."
keywords:
  - NSG
  - network security group
  - firewall rule
  - exposure
  - inbound
---

# Network security groups

A network security group (NSG) is a stateful allow-list of inbound and outbound rules bound to a subnet or a NIC. The common failure is an inbound rule with source `Internet` or `*` on a management or database port, putting RDP, SSH, or a database on the public internet. With reader access you read every NSG and resolve which resources it fronts; with `Microsoft.Network/networkSecurityGroups/securityRules/write` you add your own allow rule to reach something previously filtered.

## Finding wide-open inbound rules

```bash
# NSGs with an Allow inbound rule open to the internet
az network nsg list --query "[].{Name:name,RG:resourceGroup}" -o table
az network nsg rule list --nsg-name <nsg> -g <rg> \
  --query "[?direction=='Inbound' && access=='Allow' && (sourceAddressPrefix=='*' || sourceAddressPrefix=='Internet')].{Name:name,Port:destinationPortRange,Src:sourceAddressPrefix}" -o table
```

## Opening a path you control

```bash
# add an inbound allow rule (needs securityRules/write on the NSG)
az network nsg rule create -g <rg> --nsg-name <nsg> -n allow-attacker \
  --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes <your-ip>/32 --destination-port-ranges 3389 22 1433
```

## Exploitation notes

- `3389`, `22`, `1433`, `3306`, `5432`, `6379`, and `27017` open to `Internet` are the high-value finds: RDP, SSH, and weakly authenticated data stores.
- NSG rules apply at subnet and NIC level; a permissive subnet rule can be undone by a tighter NIC rule and vice versa, so check both.
- Adding a rule is logged in the Activity Log; a lower-priority number wins, so a new rule at priority 100 overrides a deny at 200 without deleting it.

## Tools

- **AWS CLI equivalent is `az network nsg`**: enumeration and rule writes.
- **MicroBurst** / **Azure Resource Graph**: NSG posture across the subscription.
- **nmap** / **masscan**: confirm reachability of exposed ports from outside.

## References

- [HackTricks Cloud: Azure networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: network security groups](https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
