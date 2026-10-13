---
order: 2
title: "Abus du SSO et des tokens : forger l'identité vCenter via le système SSO"
description: "Le Single Sign-On de vCenter émet des tokens SAML adossés à un certificat de signature détenu dans le vmdir/VMware Directory et le STS. Un attaquant qui atteint ce matériel de clés, ou les identifiants vpxuser et solution-user dans la base de données vCenter, forge ou réutilise des tokens pour s'authentifier en tant qu'administrateur, obtenant le contrôle total de vCenter sans mot de passe."
keywords:
  - sso vcenter
  - saml
  - sts
  - vmdir
  - vpxuser
---

# Abus du SSO et des tokens

L'authentification vCenter passe par le Single Sign-On : le Security Token Service (STS) émet des tokens SAML signés par un certificat dont la clé privée réside dans le VMware Directory embarqué (vmdir) et la configuration du STS. Les services et les administrateurs s'authentifient en présentant ces tokens. L'abus consiste à obtenir la clé de signature ou les identifiants de service stockés, puis à forger ou réutiliser des tokens, s'authentifiant en tant que `administrator@vsphere.local` ou un solution user sans jamais connaître de mot de passe. Les secrets pertinents se trouvent sur l'appliance vCenter et dans ses bases de données.

## Où réside le matériel de clés

```bash
# on the vCenter appliance: the STS/IdP signing key and certs
ls -l /etc/vmware-sso/ /storage/db/vmware-vmdir/ 2>/dev/null
# the IdP signer certificate and key used to sign SAML tokens
/usr/lib/vmware-vmafd/bin/vecs-cli entry list --store STS_INTERNAL_SSL_CERT 2>/dev/null
# vmdir holds identity data; it is an LDAP directory under /storage/db/vmware-vmdir
```

## Forger et réutiliser des tokens

```bash
# with the STS IdP signing key, mint a SAML assertion for a privileged subject
# (tools automate building a valid, signed SAML token for administrator@vsphere.local),
# then present it to the vSphere API to obtain an authenticated session
# the vpxuser credential (vCenter -> ESXi host auth) is stored in the vCenter DB,
# encrypted with a key also on the appliance; recovering it authenticates to hosts
```

Le compte `vpxuser` est la manière dont vCenter s'authentifie auprès de chaque hôte ESXi géré ; son identifiant est conservé dans la base de données vCenter, chiffré avec une clé présente sur l'appliance, de sorte qu'un attaquant disposant d'un accès à l'appliance le récupère puis se connecte directement à chaque hôte géré.

## Notes d'exploitation

- L'accès aux fichiers de l'appliance (depuis un exploit de gestion ou un root volé) est la condition préalable habituelle ; de là, la clé de signature STS permet de forger des tokens SAML admin, contournant entièrement l'authentification par mot de passe.
- Les tokens SAML forgés sont acceptés par l'API comme une session SSO valide ; il existe des outils qui assemblent et signent l'assertion à partir de la clé IdP, donnant une session administrateur.
- La récupération de `vpxuser` pivote de vCenter vers chaque hôte ESXi qu'il gère, donc la compromission du SSO se propage en cascade vers toute la flotte ; combinez avec l'[accès à l'hôte ESXi](../esxi/host-access-and-shell.md).
- Les solution users (les identités de service internes) sont de même adossés à des certificats dans vmdir ; les récupérer permet de s'authentifier en tant que services vCenter de confiance.

## Outils

- [vCenter SAML token tooling (e.g. vcenter_saml_login research)](https://github.com/horizon3ai)

## Références

- [VMware: vCenter Single Sign-On architecture](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [Horizon3: vCenter SSO/SAML research](https://www.horizon3.ai/attack-research/)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
