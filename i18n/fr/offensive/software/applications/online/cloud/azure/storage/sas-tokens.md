---
title: "Jetons SAS : détourner des signatures d'accès partagé sur-privilégiées"
order: 2
description: "Détourner les signatures d'accès partagé Azure : URL SAS account, service et user-delegation sur-privilégiées pour un accès durable aux données."
keywords:
  - SAS token
  - shared access signature
  - user delegation
  - account SAS
  - data access
---

# Jetons SAS

Une signature d'accès partagé (SAS) est une chaîne de requête signée qui accorde l'accès aux données de stockage sans connexion. Il en existe trois types : une **account SAS** (signée par la clé de compte, pouvant s'étendre à plusieurs services et opérations), une **service SAS** (signée par la clé de compte, limitée à une seule ressource) et une **user-delegation SAS** (signée par un jeton Entra, limitée au RBAC du signataire). Les URL SAS sont couramment sur-privilégiées (permissions complètes, expiration à un an) et fuitent dans le code, la configuration, les tickets et les logs, où elles deviennent un accès durable sans credential.

## Forger une SAS à partir d'une clé que vous détenez

```bash
# account SAS: all services, all resource types, broad permissions, long expiry
az storage account generate-sas --account-name acme --account-key <KEY> \
  --services bfqt --resource-types sco \
  --permissions rwdlacup --expiry 2027-01-01 -o tsv

# container service SAS
az storage container generate-sas --account-name acme --account-key <KEY> \
  -n backups --permissions rl --expiry 2027-01-01 -o tsv
```

## Utiliser une SAS trouvée

```bash
# the SAS is the whole credential; append it to the resource URL
curl "https://acme.blob.core.windows.net/backups/db.bak?<SAS>"
az storage blob list --account-name acme --container-name backups --sas-token '<SAS>'
```

## Notes d'exploitation

- Une SAS ne peut pas être révoquée individuellement, sauf si elle est liée à une stored access policy ; une SAS signée par la clé de compte reste valide jusqu'à son expiration ou une rotation de clé, de sorte qu'une SAS à longue durée de vie divulguée est un accès durable et une prise de pied de persistance.
- Décodez les champs de permission et d'expiration de toute SAS que vous trouvez (`sp=`, `se=`, `sr=`, `ss=`, `srt=`) : `sp=rwdlacup` avec un `se=` lointain correspond à un contrôle total.
- Une user-delegation SAS n'est aussi limitée que le RBAC du signataire, mais elle survit à la session de ce dernier ; une account SAS ignore totalement le RBAC.
- Passez au grep les dépôts, les variables de pipeline et les sorties de template ARM à la recherche de chaînes SAS `?sv=...&sig=`.

## Outils

- **az CLI** (`storage ... generate-sas`) : forger et utiliser des SAS.
- **MicroBurst** : fait remonter clés et SAS lors de l'énumération du stockage.
- **TruffleHog** / scanners de secrets : trouver des URL SAS divulguées dans le code source et la configuration.

## Références

- [HackTricks Cloud: Azure SAS tokens](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [Microsoft: shared access signatures](https://learn.microsoft.com/azure/storage/common/storage-sas-overview)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
