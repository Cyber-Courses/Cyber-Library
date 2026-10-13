---
order: 2
title: "Mappage de certificat : ESC9, ESC10, ESC14"
description: "Abuser du mappage faible entre certificat et compte dans Active Directory : l'extension de sécurité SID manquante (ESC9), les paramètres de registre de mappage Kerberos et Schannel faibles (ESC10), et les mappages explicites écrits par l'attaquant via altSecurityIdentities (ESC14)."
keywords:
  - ESC9
  - ESC10
  - ESC14
  - altSecurityIdentities
  - certificate mapping
---

# Mappage de certificat

Lorsqu'un compte s'authentifie avec un certificat, le contrôleur de domaine doit décider **quel compte** le certificat représente. L'extension de sécurité SID (`szOID_NTDS_CA_SECURITY_EXT`) a été introduite pour lier un certificat au SID de son propriétaire afin que le sujet ne puisse pas être usurpé. Ces techniques abusent des mappages où cette liaison est **manquante ou faible**, de sorte qu'un certificat se mappe sur un compte plus privilégié qu'il ne le devrait.

## ESC9 : pas d'extension de sécurité

Un modèle marqué `CT_FLAG_NO_SECURITY_EXTENSION` produit des certificats **sans** la liaison SID. Cela n'aide que lorsque le DC autorise encore le mappage faible : avec `StrongCertificateBindingEnforcement` à `0` (désactivé) ou `1` (compatibilité), un certificat dépourvu de l'extension SID retombe sur le mappage par **UPN**, mais à `2` (application complète), il est **rejeté** d'emblée. Sur un DC à mappage faible, si vous pouvez contrôler l'UPN d'un principal (par exemple vous avez un accès en écriture sur un utilisateur peu privilégié et pouvez définir son UPN sur celui d'une cible), vous vous inscrivez avec ce compte, définissez son UPN sur la victime, et le certificat émis se mappe sur la victime :

```bash
# With GenericWrite over 'puppet': set its UPN to the target, enrol, then restore
certipy account update -u user@example.local -p pass -user puppet -upn administrator@example.local
certipy req -u puppet@example.local -p pass2 -ca <ca> -template <esc9-template>
certipy account update -u user@example.local -p pass -user puppet -upn puppet@example.local
certipy auth -pfx administrator.pfx -domain example.local   # maps to administrator
```

## ESC10 : configuration de mappage faible

ESC10 est la même usurpation rendue possible par des paramètres de **registre** sur le DC plutôt que par un drapeau de modèle :

- **Kerberos** : `StrongCertificateBindingEnforcement = 0` désactive l'exigence de liaison SID, le mappage par UPN est donc approuvé.
- **Schannel** : `CertificateMappingMethods = 0x4` active le mappage par UPN pour Schannel.

L'un ou l'autre permet à l'astuce d'échange d'UPN ci-dessus de fonctionner contre les modèles standard, étendant ESC9 d'un seul modèle à toute la CA.

## ESC14 : mappage explicite écrit par l'attaquant

Les comptes peuvent porter des mappages de certificat **explicites** dans l'attribut `altSecurityIdentities`. Si vous disposez d'un accès en écriture sur cet attribut d'un compte cible, vous ajoutez un mappage vers un certificat que vous contrôlez, puis vous vous authentifiez en tant que la cible avec celui-ci :

```bash
# Add an explicit mapping tying your certificate to the victim, then authenticate
certipy account update -u user@example.local -p pass -user victim \
  -alt-security-identities '<mapping-string-for-your-cert>'
```

Il s'agit d'un pur abus d'ACL : une primitive d'écriture sur `altSecurityIdentities` (depuis BloodHound) devient une authentification en tant que la cible.

## Notes d'exploitation

- Ces techniques s'associent à un chemin d'inscription : ESC9 nécessite un modèle depuis lequel vous pouvez vous inscrire, ESC14 nécessite un certificat à mapper ; c'est la faiblesse de mappage qui transforme le certificat en *mauvaise* identité.
- Lorsque l'extension SID est appliquée (corrigée, liaison forte), l'usurpation naïve par SAN ([ESC1](vulnerable-templates.md)) échoue et ces lacunes de mappage ou [ESC15](vulnerable-templates.md) deviennent la voie de passage.
- La variante Schannel d'ESC10 permet l'authentification via LDAPS/HTTPS plutôt que Kerberos, utile lorsque PKINIT est contraint.

## Outils

- **Certipy** (`account update`, `req`, `auth`) : échanges d'UPN, écritures d'altSecurityIdentities, et PKINIT.
- **PowerView / LDAP natif** : définir `userPrincipalName` et `altSecurityIdentities`.

## Références

- [Certipy wiki: privilege escalation (ESC9/ESC10)](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [SpecterOps: BloodHound ADCSESC9 edge](https://bloodhound.specterops.io/resources/edges/adcs-esc9a)
- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
