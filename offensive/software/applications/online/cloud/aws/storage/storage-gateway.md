---
title: "Storage Gateway: abusing the on-premises to S3 and EBS bridge"
order: 6
description: "Abusing Storage Gateway file shares and cached volumes that bridge on-premises access to S3 and EBS."
keywords:
  - Storage Gateway
  - file share
  - volume
  - S3
  - hybrid
---

# Storage Gateway

Storage Gateway bridges on-premises hosts to AWS storage: file gateways expose S3 buckets as NFS or SMB shares, and volume gateways present EBS-backed iSCSI volumes. The gateway holds standing access to the backing S3 and EBS, so reaching a gateway's share, or the control-plane configuration of the gateway itself, is an indirect route to the cloud storage behind it.

## Enumerating gateways and shares

```bash
aws storagegateway list-gateways --query 'Gateways[].[GatewayId,GatewayType]'
aws storagegateway list-file-shares --query 'FileShareInfoList[].FileShareARN'
aws storagegateway describe-nfs-file-shares --file-share-arn-list <arn> \
  --query 'NFSFileShareInfoList[].[LocationARN,ClientList,Squash]'
aws storagegateway list-volumes
```

The `LocationARN` on a file share names the S3 bucket it fronts; the `ClientList` and squash settings tell you who may mount it.

## Reaching the data

```bash
# mount an NFS file gateway share exposed to a reachable CIDR
sudo mount -t nfs <gateway-ip>:/<bucket> /mnt/gw
```

## Exploitation notes

- A file gateway gives NFS/SMB access to the S3 bucket behind it without S3 API permissions, bypassing bucket-policy controls that assume API access.
- Volume gateway iSCSI targets with no CHAP, or weak CHAP, are mountable by anyone who can reach the gateway's network interface.
- Gateway control-plane access (`storagegateway:*`) lets you create a new share over an existing bucket, a quieter read path than touching S3 directly.

## Tools

- **AWS CLI** (`storagegateway list-/describe-*`): enumerate gateways, shares, and their backing locations.
- **mount.nfs** / **iscsiadm**: mount the exposed shares and volumes.

## References

- [HackTricks Cloud: Storage Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [AWS: Storage Gateway file shares](https://docs.aws.amazon.com/storagegateway/latest/userguide/GettingStartedCreateFileShare.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
