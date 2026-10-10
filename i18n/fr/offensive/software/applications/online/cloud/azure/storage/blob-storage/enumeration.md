---
title: "Énumération : trouver des conteneurs et comptes de stockage publics"
order: 1
description: "Découvrir les conteneurs de blobs et comptes de stockage Azure publics par devinette de noms et listage anonyme."
keywords:
  - blob enumeration
  - public container
  - storage account
  - anonymous
  - discovery
---

# Énumération

Les noms de compte de stockage sont globaux et résolubles via DNS (`<name>.blob.core.windows.net`) : l'espace de noms des comptes est donc attaquable par force brute depuis l'extérieur, sans aucun credential, et tout conteneur laissé à un niveau d'accès public liste et sert ses blobs de manière anonyme. L'énumération consiste à deviner les noms de comptes et de conteneurs, puis à confirmer le listage anonyme.

## Trouver des comptes de stockage

```bash
# resolve candidate account names (org-themed wordlists), valid = DNS answer
for n in acme acmeprod acmebackups acmedata; do
  host $n.blob.core.windows.net | grep -q address && echo "exists: $n"
done
```

MicroBurst automatise le balayage des sous-domaines et des conteneurs sur l'ensemble des endpoints de stockage (blob, file, queue, table) :

```
Invoke-EnumerateAzureSubDomains -Base acme -Verbose
Invoke-EnumerateAzureBlobs -Base acme          # guesses containers and lists public ones
```

## Confirmer un conteneur public

```bash
# anonymous container listing over REST (no auth)
curl "https://acme.blob.core.windows.net/backups?restype=container&comp=list"

# with the CLI, no credential
az storage blob list --account-name acme --container-name backups --auth-mode login
```

Un `200` accompagné d'une liste de blobs (ou d'un `EnumerationResults` XML) signifie que le conteneur est public ; un `PublicAccessNotPermitted` signifie qu'il existe mais qu'il est verrouillé.

## Notes d'exploitation

- L'accès public se définit par conteneur ; un compte peut avoir un conteneur lisible par tous à côté de conteneurs verrouillés : énumérez donc les noms de conteneurs même lorsque le compte est privé par défaut.
- `goblob`, `basacwler` et `cloud_enum` effectuent la même force brute à grande échelle contre des noms de conteneurs courants (backup, logs, config, web, public).
- Résolvez les noms candidats avant de lister : une réponse DNS confirme l'existence du compte et évite des tentatives de listage bruyantes contre des noms inexistants.

## Outils

- **MicroBurst** (`Invoke-EnumerateAzureBlobs`, `Invoke-EnumerateAzureSubDomains`) : découverte de comptes et de conteneurs.
- **cloud_enum** : force brute multi-cloud du stockage public, y compris les blobs Azure.
- **az CLI** (`storage blob list --auth-mode login`) : listage anonyme et authentifié.

## Références

- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure storage enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [cloud_enum](https://github.com/initstring/cloud_enum)
- [Microsoft: Blob service REST list containers](https://learn.microsoft.com/rest/api/storageservices/list-containers2)
