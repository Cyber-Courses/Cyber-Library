---
title: "Énumération : tenant, utilisateurs et fédération sans identifiants"
order: 1
description: "Énumérer les utilisateurs Entra, les détails du tenant et la configuration de fédération sans identifiants : GetCredentialType, sondage OneDrive et autodiscover, et reconnaissance de tenant avec AADInternals."
keywords:
  - énumération d'utilisateurs
  - énumération de tenant
  - GetCredentialType
  - AADInternals
  - fédération
---

# Énumération

Entra divulgue une quantité surprenante d'informations avant l'authentification. Le service de connexion confirme si un nom d'utilisateur existe et si son domaine est managed ou fédéré, et les métadonnées de tenant ouvertes révèlent l'ID du tenant, les domaines et le branding. Cela construit la liste des utilisateurs et vous indique quel chemin de connexion attaquer.

## Reconnaissance du tenant et des domaines

```powershell
# AADInternals: tenant details, domains, and federation status from outside
Invoke-AADIntReconAsOutsider -DomainName example.com
```

```bash
# tenant id and endpoints from the OpenID configuration
curl -s https://login.microsoftonline.com/example.com/.well-known/openid-configuration
# federation / managed status for a user
curl -s "https://login.microsoftonline.com/common/GetCredentialType" \
  -H 'Content-Type: application/json' \
  -d '{"Username":"user@example.com"}'
```

## Validation des utilisateurs

`GetCredentialType` (et le sondage OneDrive ou Autodiscover) renvoie un `IfExistsResult` qui confirme les noms d'utilisateur valides sans tentative de connexion, de sorte qu'une liste de noms est validée discrètement avant tout spray.

## Notes d'exploitation

- Un `IfExistsResult` de 0 signifie que le compte existe ; récoltez une liste validée pour alimenter le [password spraying](password-spraying.md).
- Les domaines fédérés envoient l'authentification vers un IdP externe (AD FS), ce qui change la cible du spray et le comportement de verrouillage.
- Rien de tout cela ne consomme une tentative de connexion, donc cela ne déclenche pas Smart Lockout ni les alertes.

## Outils

- **AADInternals** (`Invoke-AADIntReconAsOutsider`) : reconnaissance de tenant de l'extérieur vers l'intérieur.
- **o365spray** / **TeamFiltration** : validation de noms d'utilisateur et OSINT.

## Références

- [AADInternals : outsider recon](https://aadinternals.com/aadinternals/)
- [TeamFiltration (Flangvik)](https://github.com/Flangvik/TeamFiltration)
- [HackTricks Cloud : Azure enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
