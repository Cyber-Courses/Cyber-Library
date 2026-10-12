---
title: "Kerberos relay : réacheminer l'authentification Kerberos"
order: 10
description: "Relais de l'authentification Kerberos avec krbrelayx : capture de TGT via la délégation non contrainte, et relais de tickets de service Kerberos forcés vers LDAP ou AD CS lorsque l'attaquant contrôle le nom de service auprès duquel la victime s'authentifie."
keywords:
  - kerberos relay
  - krbrelayx
  - unconstrained delegation
  - AD CS
  - SPN
---

# Relay

Le [relais NTLM](../ntlm/relay.md) fonctionne parce que NTLM n'est pas lié à une cible. Kerberos est plus difficile à relayer car un ticket de service est **chiffré pour un SPN précis**, de sorte que vous ne pouvez pas simplement réacheminer une authentification vers un service arbitraire. **krbrelayx** résout cela de deux façons : il capture des TGT via la **délégation non contrainte**, et, là où vous pouvez amener une victime à s'authentifier auprès d'un **nom de service que vous contrôlez**, il relaie cette authentification Kerberos vers l'avant.

## Capture par délégation non contrainte

Le mode d'origine de krbrelayx est le pendant de la [délégation non contrainte](delegation/unconstrained.md) : un hôte approuvé pour la délégation non contrainte reçoit les **TGT** des appelants, et krbrelayx, muni de la clé Kerberos de cet hôte, déchiffre l'authentification entrante et extrait le TGT pour le réutiliser :

```bash
# Provide the unconstrained-delegation host's AES key or hash; krbrelayx extracts coerced TGTs
krbrelayx.py -aesKey <host-aes-key>
# then coerce a DC to authenticate to this host and capture DC$'s TGT
```

## Relais vers le nom de service propre à la cible

Vous ne pouvez pas relayer un ticket construit pour un SPN que vous possédez : il est chiffré pour la clé de votre compte, la vraie cible le rejette donc. krbrelayx exige à la place que le **nom d'hôte du ticket entrant corresponde à la cible du relais**, et vous amenez la victime à s'authentifier auprès du nom de la *cible* en **redirigeant la résolution de noms** :

- Faites pointer le nom d'hôte propre à la cible vers votre écouteur avec [ADIDNS](../ntlm/adidns.md) ou `mitm6`, puis forcez la victime ; elle demande un ticket pour le SPN de la vraie cible mais envoie l'authentification vers vous, et krbrelayx la réachemine vers la cible réelle.
- Relayez cette authentification Kerberos vers **LDAP** (pour RBCD ou shadow credentials) ou vers l'enrôlement web **AD CS** (ESC8) :

```bash
# Relay coerced Kerberos auth to AD CS web enrolment for a certificate
krbrelayx.py --target 'http://<ca>/certsrv/' --adcs --template Machine
```

## Notes d'exploitation

- Le relais Kerberos contourne le durcissement NTLM : là où NTLM est désactivé ou que la signature SMB/LDAP bloque le relais NTLM, un chemin Kerberos peut toujours atteindre LDAP ou AD CS.
- La contrainte est le **nom d'hôte du SPN** du ticket : il doit correspondre à la cible du relais, vous redirigez donc le nom de la cible vers votre écouteur avec [ADIDNS](../ntlm/adidns.md)/`mitm6` plutôt que de posséder le SPN, et vous couplez cela avec de la [coercition](../ntlm/coercion.md).
- La capture par délégation non contrainte reste l'usage le plus fiable de krbrelayx : forcez un DC vers un hôte de délégation que vous possédez et récupérez le TGT de `DC$`, puis faites un DCSync.

## Outils

- **krbrelayx.py** (dirkjanm) : capture de TGT via la délégation non contrainte et relais Kerberos vers HTTP/LDAP/AD CS.
- **dnstool.py / mitm6** : contrôler la résolution de noms pour que la victime cible votre SPN.
- **Coercer / PetitPotam** : source de l'authentification.

## Références

- [krbrelayx (dirkjanm)](https://github.com/dirkjanm/krbrelayx)
- [dirkjanm: relaying Kerberos over DNS with krbrelayx and mitm6](https://dirkjanm.io/relaying-kerberos-over-dns-with-krbrelayx-and-mitm6/)
- [Synacktiv: relaying Kerberos over SMB using krbrelayx](https://www.synacktiv.com/en/publications/relaying-kerberos-over-smb-using-krbrelayx)
